---
title: "Building an AI Agent for Network Automation: Talk to Your Router Like a Human"
date: 2025-11-14 13:00:00 +0500
categories: miscellaneous
tags:
  - langchain
  - groq
  - ai-coding
  - python
  - devops
---

A Python application that lets you talk to Cisco routers in plain English. Instead of memorizing commands and parsing outputs, you ask questions naturally:

- "Show me all interfaces and their status"
- "What's the device uptime?"
- "Are there any errors?"

The AI figures out which commands to run, executes them, and gives you clear answers.

---

## Technology Stack

**Three core components:**

1. **Netmiko** - SSH connection to network devices
2. **LangChain** - AI application framework
3. **Groq** - Fast, free AI inference (Llama 3.3)

---

## Prerequisites

```bash
# Python 3.12 or higher
# Install uv package manager
curl -LsSf https://astral.sh/uv/install.sh | sh

# Get free Groq API key
# Visit: https://console.groq.com/keys
```

---

## Project Setup

### 1. Create Project Directory

```bash
mkdir network-ai-agent
cd network-ai-agent
```

### 2. Create `pyproject.toml`

```toml
[project]
name = "network-agent"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "groq>=0.33.0",
    "langchain-groq>=1.0.0",
    "langgraph>=1.0.3",
    "netmiko>=4.6.0",
    "python-dotenv>=1.2.1",
]
```

### 3. Create `.env` File

```bash
GROQ_API_KEY=your_groq_api_key_here
DEVICE_PASSWORD=your_device_password  # Optional
```

### 4. Install Dependencies

```bash
uv sync
```

---

## The Complete Code

Create `main.py`:

```python
"""
AI Agent for Network Devices
Talk to your Cisco router using natural language
"""

import os
import getpass
from dotenv import load_dotenv
from netmiko import ConnectHandler
from langchain_groq import ChatGroq
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent


class NetworkAgent:
    """AI-powered network device assistant."""

    def __init__(self, groq_api_key: str):
        """Initialize the AI agent."""
        # Set up the AI model
        self.llm = ChatGroq(
            groq_api_key=groq_api_key,
            model_name="llama-3.3-70b-versatile",
            temperature=0.7
        )
        self.connection = None

        # Create a tool the AI can use to run commands
        self.execute_command_tool = tool("execute_show_command")(
            self._execute_command
        )

        # Create the AI agent with the tool
        self.agent = create_react_agent(self.llm, [self.execute_command_tool])

    def connect(self, hostname: str, username: str, password: str):
        """Connect to a network device."""
        device_config = {
            'device_type': 'cisco_ios',
            'host': hostname,
            'username': username,
            'password': password,
            'timeout': 30,
        }
        self.connection = ConnectHandler(**device_config)
        print(f"✓ Connected to {hostname}")

    def disconnect(self):
        """Disconnect from the device."""
        if self.connection:
            self.connection.disconnect()
            print("✓ Disconnected")

    def _execute_command(self, command: str) -> str:
        """Execute a command on the device (used by AI)."""
        if not self.connection:
            return "Error: Not connected to device"

        try:
            output = self.connection.send_command(command)
            return output
        except Exception as e:
            return f"Error: {str(e)}"

    def ask(self, question: str) -> str:
        """Ask the AI a question about the device."""
        # Give the AI instructions
        prompt = f"""You are a network engineer assistant.
Use the execute_show_command tool to run Cisco commands.
Answer the user's question clearly and concisely.

User question: {question}"""

        try:
            result = self.agent.invoke({"messages": [("user", prompt)]})

            # Extract the AI's response
            last_message = result["messages"][-1]
            return last_message.content
        except Exception as e:
            return f"Error: {str(e)}"


def main():
    """Main program."""
    load_dotenv()

    print("="*60)
    print("AI Network Agent")
    print("="*60)

    # Get connection details
    hostname = input("\nDevice IP: ").strip()
    username = input("Username: ").strip()
    password = os.getenv('DEVICE_PASSWORD') or getpass.getpass("Password: ")

    # Get API key
    api_key = os.getenv('GROQ_API_KEY')
    if not api_key:
        print("Error: Set GROQ_API_KEY in .env file")
        return

    # Create and connect agent
    agent = NetworkAgent(api_key)

    try:
        agent.connect(hostname, username, password)

        print("\n" + "="*60)
        print("Ready! Type 'quit' to exit")
        print("="*60 + "\n")

        # Chat loop
        while True:
            question = input("\n💬 Ask: ").strip()

            if question.lower() in ['quit', 'exit']:
                break

            if not question:
                continue

            print("\n" + "-"*60)
            answer = agent.ask(question)
            print(answer)
            print("-"*60)

    except Exception as e:
        print(f"Error: {e}")
    finally:
        agent.disconnect()


if __name__ == "__main__":
    main()
```

---

## How It Works: Code Breakdown

### 1. Imports and Setup

```python
import os
import getpass
from dotenv import load_dotenv
from netmiko import ConnectHandler
from langchain_groq import ChatGroq
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent
```

**What each does:**

- `os`, `getpass` - Read environment variables and secure password input
- `load_dotenv` - Load API keys from `.env` file
- `ConnectHandler` - Create SSH sessions with Cisco devices
- `ChatGroq` - Connect to Llama 3.3 on Groq's platform
- `tool` - Expose Python functions to the AI
- `create_react_agent` - Build an agent that can plan and execute actions

### 2. The NetworkAgent Class

**Constructor (`__init__`)**

```python
self.llm = ChatGroq(
    groq_api_key=groq_api_key,
    model_name="llama-3.3-70b-versatile",
    temperature=0.7
)
```

Creates the AI model connection. Temperature 0.7 balances creativity with consistency.

```python
self.execute_command_tool = tool("execute_show_command")(self._execute_command)
```

Wraps the `_execute_command` method as a tool the AI can call.

```python
self.agent = create_react_agent(self.llm, [self.execute_command_tool])
```

Creates an agent that can:

- Reason about questions
- Decide when to use tools
- Analyze command outputs
- Generate answers

### 3. Connection Management

**`connect()` method:**

```python
device_config = {
    'device_type': 'cisco_ios',
    'host': hostname,
    'username': username,
    'password': password,
    'timeout': 30,
}
self.connection = ConnectHandler(**device_config)
```

Builds SSH configuration and opens a live session stored in `self.connection`.

**`disconnect()` method:**

```python
if self.connection:
    self.connection.disconnect()
```

Cleanly closes the SSH session to prevent stuck connections.

### 4. The Command Execution Tool

```python
def _execute_command(self, command: str) -> str:
    output = self.connection.send_command(command)
    return output
```

This is the bridge between AI and device. When the AI needs information:

- Runs the CLI command through Netmiko
- Returns raw output
- The agent analyzes this output to form a reply

### 5. The Question Handler

```python
def ask(self, question: str) -> str:
    prompt = f"""You are a network engineer assistant.
Use the execute_show_command tool to run Cisco commands.
Answer the user's question clearly and concisely.

User question: {question}"""
```

Constructs a prompt that:

- Instructs the AI to use the tool
- Encourages clear, concise replies
- Includes the user's question

```python
result = self.agent.invoke({"messages": [("user", prompt)]})
last_message = result["messages"][-1]
return last_message.content
```

The agent flow:

1. Reads your question
2. Plans which command to run
3. Calls `_execute_command` tool
4. Analyzes CLI output
5. Returns final answer

### 6. The Main Function

**Load environment:**

```python
load_dotenv()
```

Loads `GROQ_API_KEY` and optional `DEVICE_PASSWORD`.

**Get credentials:**

```python
hostname = input("Device IP: ").strip()
username = input("Username: ").strip()
password = os.getenv('DEVICE_PASSWORD') or getpass.getpass("Password: ")
```

Pulls password from `.env` or prompts securely.

**Chat loop:**

```python
while True:
    question = input("\n💬 Ask: ").strip()
    if question.lower() in ['quit', 'exit']:
        break
    answer = agent.ask(question)
    print(answer)
```

Continuously processes questions until user types "quit".

**Cleanup:**

```python
finally:
    agent.disconnect()
```

Always closes connection, even on errors.

---

## The Complete Flow

```
User: "Show me all interfaces"
  ↓
AI Agent receives question
  ↓
AI thinks: "I need 'show ip interface brief'"
  ↓
AI calls: execute_show_command("show ip interface brief")
  ↓
Netmiko runs command via SSH
  ↓
Device returns CLI output
  ↓
AI analyzes output
  ↓
AI responds: "Here are your interfaces..."
```

---

## Running Your Agent

```bash
uv run main.py
```

**Example session:**

```
Device IP: 192.168.1.1
Username: admin
Password: ****
✓ Connected to 192.168.1.1

Ready! Type 'quit' to exit

💬 Ask: Show me all interfaces

------------------------------------
I found 4 interfaces on your device:
1. GigabitEthernet0/0 - UP (192.168.1.1)
2. GigabitEthernet0/1 - UP (10.1.0.1)
3. GigabitEthernet0/2 - DOWN
4. Loopback0 - UP (10.0.0.1)
------------------------------------

💬 Ask: What's the uptime?

------------------------------------
The device has been running for 2 days, 4 hours, 30 minutes.
------------------------------------

💬 Ask: quit
✓ Disconnected
```

---

## Example Queries

**Basic Information:**

- "What version is running?"
- "What's the hostname?"
- "Show me the uptime"

**Interface Management:**

- "List all interfaces"
- "Which interfaces are down?"
- "Show me interface errors"
- "What's the status of GigabitEthernet0/1?"

**Routing:**

- "Show me the routing table"
- "What's the default gateway?"
- "Show me all static routes"

**Troubleshooting:**

- "Are there any errors in the logs?"
- "Show me interface errors"
- "Is there any packet loss?"

---

## Traditional vs AI Approach

### Traditional Way

```
You: show ip interface brief
Device: [50 lines of output]
You: [manually read through output]
You: show interfaces status
Device: [more output]
You: [correlate information mentally]
```

### AI Agent Way

```
You: "Which interfaces have errors?"
AI: [runs multiple commands automatically]
AI: [analyzes all output]
AI: "Interface GigabitEthernet0/2 has 45 input errors"
```

---

## Security Best Practices

🔒 **Critical security considerations:**

1. **Never hardcode passwords** - Use environment variables or secure input
2. **Use SSH keys** when possible instead of passwords
3. **Secure API keys** - Store in `.env`, never commit to version control
4. **Read-only commands** - This example only runs `show` commands
5. **Network segmentation** - Run from secure management network
6. **Audit logging** - Track which commands are executed

---

## Troubleshooting

### "Connection timeout"

- Verify device IP is reachable
- Check SSH is enabled on device
- Confirm firewall rules allow SSH
- Test with manual SSH first

### "Authentication failed"

- Verify username and password
- Check if enable password is required
- Confirm privilege level is sufficient

### "API rate limit"

- Groq free tier: 30 requests/minute
- Wait between queries
- Consider paid tier for production use

### "Command not recognized"

- Commands differ by platform (IOS vs NX-OS vs IOS-XR)
- Try the direct command manually first
- Check device capabilities

---

## Architecture Overview

```
┌─────────────┐
│    User     │
│   (Human)   │
└──────┬──────┘
       │ "Show me interfaces"
       ↓
┌─────────────┐
│  AI Agent   │ ← Thinks & Plans
│  (Llama)    │ ← LangGraph orchestration
└──────┬──────┘
       │ execute_show_command()
       ↓
┌─────────────┐
│  Netmiko    │ ← SSH Connection
│  (Python)   │
└──────┬──────┘
       │ CLI Commands
       ↓
┌─────────────┐
│   Router    │
│  (Cisco)    │
└─────────────┘
```

---

## Enhancement Ideas

Ready to take it further? Consider:

1. **Multi-platform support** - Add NX-OS, IOS-XR, ASA
2. **Configuration changes** - Add tools for config commands (with safety checks)
3. **Multi-device queries** - Connect to multiple devices simultaneously
4. **Web interface** - Build a Flask/FastAPI web UI
5. **Scheduled checks** - Automated health monitoring
6. **Alerting system** - Notifications for critical issues
7. **Conversation memory** - Remember context across questions
8. **Command history** - Log all executed commands
9. **Output caching** - Cache recent command outputs

---

## Why This Matters

**100 lines of Python** gives you an agent that:

- Understands natural language
- Executes network commands intelligently
- Analyzes device output
- Responds with clear answers
- Eliminates command memorization
- Reduces parsing time

This is network automation meeting AI. No more CLI reference guides, no more manual output parsing - just ask and receive answers.

---

## Quick Start Checklist

- [ ] Install Python 3.12+
- [ ] Install uv package manager
- [ ] Get Groq API key from <https://console.groq.com>
- [ ] Create project directory
- [ ] Copy `pyproject.toml`
- [ ] Create `.env` with API key
- [ ] Copy `main.py`
- [ ] Run `uv sync`
- [ ] Run `uv run main.py`
- [ ] Start chatting with your devices

---

## Conclusion

You now have a working AI network agent. The combination of LangChain's agent framework, Groq's fast inference, and Netmiko's device connectivity creates a powerful tool for network operations. Customize it for your environment, add more tools, and watch your network management transform.

Happy automating! 🚀🤖
