---
title: "Building Network Automation Agent: Talk to Your Router Like a Human"
date: 2025-11-14 13:00:00 +0500
categories: miscellaneous
tags:
  - langchain
  - groq
  - python
  - devops
  - nemiko
---

This guide explains how to build a simple AI agent that understands plain language and runs network commands for you. It uses LangGraph for workflow control and Netmiko for device access. All code blocks stay the same as before.

---

## Introduction

Managing networks usually means opening many SSH sessions and typing long commands. It can take time and lead to mistakes. What if you could just type:

**“Show VLANs on switch‑1”** and the ai agent handles everything?

With LLMs, this is now possible. By combining a language model with a workflow and a device‑access tool, you can build an agent that understands what you want and runs the correct command.

In this guide, you will learn:

* How the agent works
* How to set up the project
* How the workflow processes user messages
* How commands run on real network devices

---

## Section 1: Understanding the Agent Design

The agent works like this:

1. It reads your message and figures out your request.
2. It checks if a command needs to run.
3. It connects to the device.
4. It returns an easy‑to‑read answer.

LangGraph is used to build a small state machine with three parts:

* **Understand**: detects intent and devices
* **Execute**: runs the network command
* **Respond**: creates the final reply

```
+-----------------+      +-----------------+      +-----------------+
|   Understand    +---->     Execute     +---->     Respond        |
+-----------------+      +-----------------+      +-----------------+
```

### State Machine Code

```python
"""Defines the state graph for the network automation agent."""

from typing import TypedDict, List, Any

from langgraph.graph import END, StateGraph
from langchain_core.messages import BaseMessage

from graph.nodes import execute_node, respond_node, understand_node


class State(TypedDict):
    """Defines the state structure for the LangGraph workflow."""
    messages: List[BaseMessage]
    results: dict[str, Any]


def create_graph():
    """Creates and configures the LangGraph workflow for the network agent."""
    workflow = StateGraph(State)

    workflow.add_node("understand", understand_node)
    workflow.add_node("execute", execute_node)
    workflow.add_node("respond", respond_node)

    workflow.set_entry_point("understand")
    # Conditional edge from 'understand' node: if the LLM generated tool calls,
    # route to 'execute' node; otherwise, route directly to 'respond' node
    workflow.add_conditional_edges(
        "understand",
        lambda s: "execute"
        if hasattr(s["messages"][-1], "tool_calls") and s["messages"][-1].tool_calls
        else "respond",
        {"execute": "execute", "respond": END},
    )
    workflow.add_edge("execute", "respond")
    workflow.add_edge("respond", END)

    return workflow.compile()
```

---

## Section 2: Setting Up the Project

A common use case is checking version, uptime, or status of devices without logging into each one. Follow these steps to set up the project.

### Step 1: Clone the Repo

```bash
git clone https://github.com/sydasif/network-automation-agent.git
cd network-agent
```

### Step 2: Install Dependencies

```bash
uv sync
```

### Step 3: Add Environment Variables

```bash
cp .env.example .env
```

Add your Groq API key inside the `.env` file.

### Step 4: Add Your Devices

```yaml
# Network device inventory
devices:
  - name: s1
    host: 192.168.121.101
    username: admin
    password: admin
    device_type: cisco_ios
  - name: s2
    host: 192.168.121.102
    username: admin
    password: admin
    device_type: cisco_ios
```

---

## Section 3: How the "Understand" Node Works

This part reads the user message and decides if a device command should run. It also knows the device names so it can check if the user is referring to a valid device.

```python
"""Defines the node functions for the network automation agent workflow."""

from typing import List, Any, TypedDict

from langchain_core.messages import AIMessage, HumanMessage, SystemMessage, BaseMessage, ToolMessage

from llm.setup import create_llm
from tools.run_command import run_command
from utils.devices import load_devices

llm = create_llm()
llm_with_tools = llm.bind_tools([run_command])


# Cache for device names to avoid repeated loading in understand_node
_CACHED_DEVICE_NAMES: list[str] | None = None


def clear_device_cache():
    """Clear the cached device names to force reloading from configuration."""
    global _CACHED_DEVICE_NAMES
    _CACHED_DEVICE_NAMES = None


def understand_node(state: dict[str, Any]) -> dict[str, Any]:
    """Processes user input and determines if network commands need to be executed."""
    global _CACHED_DEVICE_NAMES

    messages: list[BaseMessage] = state.get("messages", [])

    # Use cached device names if available, otherwise load and cache them
    if _CACHED_DEVICE_NAMES is None:
        devices = load_devices()  # This will use the cached version from utils/devices
        _CACHED_DEVICE_NAMES = list(devices.keys())

    system_msg = SystemMessage(
        content=(
            "You are a network automation assistant.\n"
            f"Available devices: {', '.join(_CACHED_DEVICE_NAMES)}\n"
            "If user asks to run show commands, call run_command tool."
        )
    )

    full_messages = [system_msg]
    for m in messages:
        if isinstance(m, str):
            full_messages.append(HumanMessage(content=m))
        else:
            full_messages.append(m)

    response = llm_with_tools.invoke(full_messages)

    return {"messages": messages + [response], "results": state.get("results", {})}
```

---

## Section 4: How the "Execute" Node Works

This part connects to the device using Netmiko and runs the command. It also tries structured output first and falls back to raw output if needed.

```python
def should_execute_tools(state: dict[str, Any]) -> str:
    """Determines if the workflow should execute tools or respond directly."""
    last = state["messages"][-1]
    if hasattr(last, "tool_calls") and last.tool_calls:
        return "execute"
    return "respond"


def execute_node(state: dict[str, Any]) -> dict[str, Any]:
    """Executes network commands on specified devices based on tool calls."""
    messages = state["messages"]
    last = messages[-1]

    tool_results = []
    if hasattr(last, "tool_calls"):
        for tool_call in last.tool_calls:
            if tool_call["name"] == "run_command":
                # tool_call["args"] is the dict of arguments the tool expects
                result = run_command.invoke(tool_call["args"])
                tool_results.append({"tool_call_id": tool_call["id"], "output": result})

    # Create ToolMessage objects to pass results back to the LLM
    # Pre-allocate list with known size to avoid dynamic resizing
    tool_messages = []
    tool_messages_append = tool_messages.append  # Cache the append method
    for tr in tool_results:
        tool_messages_append(
            ToolMessage(content=str(tr["output"]), tool_call_id=tr["tool_call_id"])
        )

    return {"messages": messages + tool_messages, "results": state.get("results", {})}
```

The final part of the workflow is the "respond" node, which formats and returns the final response to the user:

```python
def respond_node(state: dict[str, Any]) -> dict[str, Any]:
    """Formats and returns the final response to the user."""
    messages = state["messages"]
    last = messages[-1]

    if isinstance(last, AIMessage) and not hasattr(last, "tool_calls"):
        return state

    synthesis_prompt = SystemMessage(
        content=(
            "Analyze the command results and provide a concise summary. "
            "If structured, prefer tables. If raw, extract key lines."
        )
    )

    response = llm.invoke(messages + [synthesis_prompt])

    return {"messages": messages + [response], "results": state.get("results", {})}
```

---

## Section 5: Full Working Example

The main script below starts the agent, receives input, and prints the agent's answer.

```python
"""Main entry point for the Network AI Agent."""

from typing import Any

from langchain_core.messages import AIMessage, BaseMessage

from graph.router import create_graph, State


def main():
    """Initialize and run the Network AI Agent in interactive mode."""
    app = create_graph()

    print("🤖 Network AI Agent Ready!")
    print("Type 'quit' to exit.\n")

    conversation_history: list[BaseMessage] = []

    while True:
        try:
            user_input = input("You: ").strip()
            if user_input.lower() in ["quit", "exit", "q"]:
                print("Goodbye!")
                break

            if not user_input:
                continue

            conversation_history.append(user_input)

            result = app.invoke({"messages": conversation_history, "results": {}})

            final_message = result["messages"][-1]
            if isinstance(final_message, AIMessage):
                response_text = final_message.content
            else:
                response_text = str(final_message)

            print(f"\n🤖 Agent: {response_text}\n")

            conversation_history = result["messages"]  # type: ignore

        except KeyboardInterrupt:
            print("\nGoodbye!")
            break
        except Exception as e:
            print(f"Error: {e}")


if __name__ == "__main__":
    main()
```

---

## Section 6: Network Command Tool

The agent uses a specialized tool for executing commands on network devices. Here's the implementation:

```python
"""Network command execution tool for the network automation agent."""

import json
from concurrent.futures import ThreadPoolExecutor, as_completed
from typing import List, Union, Dict, Any

from langchain_core.tools import tool
from netmiko import ConnectHandler
from netmiko.exceptions import NetmikoAuthenticationException, NetmikoTimeoutException

from utils.devices import load_devices, DeviceConfig


@tool
def run_command(device: str | list[str], command: str) -> str:
    """Execute a command on one or more network devices."""
    all_devices = load_devices()

    if isinstance(device, str):
        device_list = [device]
    else:
        device_list = device

    # Cache the available devices list to avoid repeated calls to .keys()
    available_devices = list(all_devices.keys())

    # Validate that all requested devices exist
    for dev in device_list:
        if dev not in all_devices:
            return json.dumps({
                "error": f"Device '{dev}' not found",
                "available_devices": available_devices,
            })

    results = {}

    def execute_on_device(dev_name: str) -> tuple[str, Dict[str, Any]]:
        """Helper function to execute a command on a single device."""
        cfg: DeviceConfig = all_devices[dev_name]
        try:
            # Establish SSH connection to the device
            conn = ConnectHandler(
                device_type=cfg["device_type"],
                host=cfg["host"],
                username=cfg["username"],
                password=cfg["password"],
                timeout=30,
            )

            try:
                # Attempt to execute command with textfsm parsing for structured output
                out = conn.send_command(command, use_textfsm=True)
                if isinstance(out, str):
                    # If output is a string, it means textfsm parsing failed or wasn't applicable
                    parsed_type = "raw"
                    parsed_output = out
                else:
                    # If output is not a string (typically a list of dicts), textfsm parsing worked
                    parsed_type = "structured"
                    parsed_output = out
            except Exception:
                # If textfsm parsing fails, execute command without parsing
                out = conn.send_command(command)
                parsed_type = "raw"
                parsed_output = out

            # Close the SSH connection
            conn.disconnect()

            return dev_name, {
                "success": True,
                "type": parsed_type,  # Indicates whether output is structured or raw
                "data": parsed_output,
            }

        except NetmikoAuthenticationException as e:
            # Handle authentication-specific errors
            return dev_name, {"success": False, "error": f"Authentication failed: {str(e)}"}
        except NetmikoTimeoutException as e:
            # Handle timeout-specific errors
            return dev_name, {"success": False, "error": f"Connection timeout: {str(e)}"}
        except Exception as e:
            # Return error information if connection or command execution fails
            return dev_name, {"success": False, "error": f"Connection error: {str(e)}"}

    # Execute commands in parallel across multiple devices using ThreadPoolExecutor
    with ThreadPoolExecutor(max_workers=min(len(device_list), 10)) as ex:
        # Submit tasks for each device to the thread pool
        futures = {ex.submit(execute_on_device, d): d for d in device_list}
        # Process completed tasks as they finish
        for fut in as_completed(futures):
            dev, out = fut.result()
            results[dev] = out

    return json.dumps({"command": command, "devices": results}, indent=2)
```

Run the script:

```bash
uv run main.py
```

---

## Section 7: Comparing With Other Methods

| Method         | Pros                            | Cons                        | Best Use Case                       |
| -------------- | ------------------------------- | --------------------------- | ----------------------------------- |
| AI Agent       | Simple to use, natural language | Needs an LLM                | Quick checks, helping new engineers |
| Ansible/Nornir | Strong for config changes       | Requires YAML or Python     | Large changes across many devices   |
| Manual CLI     | Total control                   | Time‑consuming, error‑prone | One‑off checks                      |

---

## Section 8: Common Issues

**GROQ_API_KEY missing**

* Add it inside `.env`.

**Device not found**

* Check the device name in `hosts.yaml`.

**Auth or timeout errors**

* Check credentials and reachability.

---

## Conclusion

You now have a simple, working setup for a conversational network automation agent. This approach lets you manage devices using plain language instead of writing scripts. You can extend this agent by adding more tools, device types, or custom checks.
