---
title: "Build Your Own AI Coding Assistant with Aider and Ollama Cloud"
date: 2025-10-28 13:00:00 +0500
categories: miscellaneous
tags:
  - aider
  - ollama
  - ai-coding
  - python
  - devops
description: "Learn how to set up a personal AI coding assistant using Aider and Ollama. Combine local and cloud AI models for powerful, flexible coding automation on Linux."
keywords: "Aider, Ollama, AI coding assistant, AI for developers, Python automation, local AI models, cloud AI models, network automation tools, AI DevOps setup"
---

Looking for a **free, private AI coding assistant** that integrates with your workflow?
Combine **Aider** and **Ollama** to build your own AI pair programmer that works **locally or in the cloud** — no subscriptions, full control, and perfect for automation or network engineering tasks.

---

## 🚀 Why Use Aider with Ollama?

* **Aider**: Chat directly with your Git repo. It writes, edits, and commits code automatically.
* **Ollama**: Runs **AI models locally** or connects to **cloud-hosted models** for bigger workloads.

Together, they give you the **flexibility of local processing** with the **power of cloud AI**, ideal for developers and network engineers working on Python, DevOps, or automation scripts.

---

## ⚙️ Quick [Installation](https://aider.chat/docs/install.html#installation) Guide

### Install Aider

```bash
python -m pip install aider-install
aider-install
````

### Install Ollama

Download from [ollama.com](https://ollama.com) and start it:

```bash
ollama serve
```

### Pull AI Models

```bash
ollama pull qwen2.5-coder        # Local model
ollama pull qwen3-coder:480b-cloud  # Cloud model
```

> During setup of Ollama Cloud models, if you get any error, see: <https://ollama.com/blog/cloud-models>

### Set Ollama API

```bash
export OLLAMA_API_BASE=http://127.0.0.1:11434 # Mac/Linux
setx   OLLAMA_API_BASE http://127.0.0.1:11434 # Windows, restart shell after setx
```

### Run Aider with Cloud Model

```bash
aider --model ollama_chat/qwen3-coder:480b-cloud  # Use ollama_chat instead of ollama
```

---

## 💡 Example: Generate a Python Script

Ask Aider:

> “Create a Python script using Netmiko to check Cisco interface status.”

Aider writes the script, commits it to Git, and you can run it instantly:

```bash
python interface_check.py
```

Perfect for **network automation** and **device configuration tasks**.

---

## 🧠 Essential Aider Commands

* `/add <file>` – Add a file to context
* `/undo` – Undo the last change
* `/diff` – View recent edits
* `/run <cmd>` – Run shell commands

These make Aider a smart, terminal-based coding companion.

---

## 🧩 Tips for Network Engineers

* Use **local AI models** for sensitive or offline work.
* Use **cloud models** for large scripts or complex logic.
* Be specific in prompts — clear instructions yield cleaner code.
* Set up a bash alias for different models

---

## 🔚 Final Thoughts

The **Aider + Ollama** combo gives you:

* Privacy with local models
* Power with cloud-based AI
* Zero vendor lock-in

Build your own **AI coding assistant** that grows with your skills — perfect for developers, DevOps, and network automation professionals.
