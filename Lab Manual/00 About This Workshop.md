---
author: Bronwen Aker
updated: 2026-07-10
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

# Welcome

Thank you for signing up for *Keeping Things Local: Build It, Mesh It, Lock It* at the Antisyphon AI Summit, 2026. I'm your host, Bronwen Aker, aka Corvus, aka The Cybrarian.

Large language models are powerful tools, but they come with a tradeoff: send your data to OpenAI, Anthropic, Microsoft, or Google to get results. For security practitioners, penetration testers, and anyone handling sensitive or client data, that's a problem. This workshop shows you how to build a working alternative: a private AI service running entirely on hardware you control, accessible securely across your team or engagement, with authentication built in. No data leaves your environment. No third-party dependencies. Just your AI, your rules.

# What to Expect

This workshop builds a complete service layer around a local LLM. You start with Ollama on a single VM, then expand to a two-VM setup connected over an encrypted mesh network, then lock it down with authentication. Each section builds on the previous one.

![[heartofgold_marvin_architecture.png]]
### Tools We Will be Using

* VMware Virtual Machine (Workstation or Fusion): two Linux VMs
* Ollama: local LLM runtime, pre-installed on server VM
* Selected LLM models: pre-loaded to avoid bandwidth issues
* Tailscale: encrypted mesh networking to connect the two VMs securely
* Headscale (optional): self-hosted coordination server, for running the mesh's control plane on your own hardware
* nginx: reverse proxy with basic authentication in front of Ollama

### Lessons

### [[01 What is an LLM]]
Brief overview of LLM history, capabilities, and limitations. Fundamental terminology and concepts you need for the rest of the workshop.

### [[02 Setting Up Your VMs]]
Import and configure two virtual machines: one will run Ollama (the server), the other will access it via Tailscale (the client). Basic VM networking and access.

### [[03 Working with Ollama]]
Install Ollama, confirm it's running, and pull and run models from the command line, plus a quick reference of the commands you'll use most.

### [[04 Model Customization with Modelfiles]]
Build custom models using Modelfiles. Create purpose-specific assistants: system prompts, parameter tuning, and specialized behavior.

### [[05 Tailscale Mesh Networking]]
Set up a free Tailscale account, install Tailscale on both VMs, authenticate them into an encrypted mesh, and verify that the client VM can reach Ollama on the server VM over the mesh network. This is the foundation for secure, team-accessible AI services.

### [[05b Self-Hosting the Mesh with Headscale]]
Optional advanced track. Run the coordination server yourself with Headscale instead of relying on Tailscale's hosted control plane, keeping the entire mesh on hardware you control. Same client, same downstream steps.

### [[06 Locking It Down with nginx]]
Put an nginx reverse proxy in front of Ollama with basic authentication. Now your service is remote, encrypted, and access-controlled.

### [[07 Wrap Up and Q&A]]
Data sovereignty principles, attack surface considerations, and hardening ideas for production deployments.

### [[08 References]]
Supplemental material not essential to the hands-on lab, including the full history of AI.

---

[[01 What is an LLM]] - Next >
