---
author: Bronwen Aker
venue: Antisyphon AI Summit
date: 2026-08-17
updated: 2026-07-01
presentation_type: Workshop
duration: 4 hours
---

```table-of-contents
```
## Title
Keeping Things Local: Build It, Mesh It, Lock It

## Description
What good is a private AI if you can only use it from the keyboard it runs on? This four hour workshop takes security practitioners past simply installing Ollama and into building something bigger: a locally hosted model your whole team can reach. You'll customize models for your own workflows, connect systems over an encrypted mesh network, and put an authentication layer in front of everything. Because it all runs on hardware you control, sensitive data never leaves your environment while access expands beyond a single desk. Show up with a laptop that can run two virtual machines, and walk out with a working private service you built yourself.

## What You'll Learn
By the end of the workshop, you will be able to:
- Install and configure Ollama on Linux and manage local models from the command line
- Configure Ollama to listen on the network so other systems can use it
- Customize model behavior with Modelfiles to build purpose specific assistants for your workflows
- Build an encrypted mesh network with Tailscale and connect multiple machines to it
- Put an nginx reverse proxy with basic authentication in front of Ollama to control who can reach it
- Explain the data sovereignty case for local AI and the attack surface you take on when you expose an LLM service
## System Requirements
VMware and the lab VMs are provided. Students need a laptop that can run two virtual machines at the same time:

- 16 GB RAM minimum (24 GB or more recommended)
- 60 GB free disk space
- CPU with virtualization support enabled in BIOS/UEFI
- Reliable internet connection (required for model downloads and Tailscale)

## VM / Lab / Student Information

VMware and two Linux-based lab VMs will be provided for download before class, along with import instructions and credentials. Download and import everything ahead of time; model files are large and take time to download and extract. All labs are hands on and build on each other, ending with a working, authenticated, network accessible private AI service. Setting up a free Tailscale account is part of the lab, so no advance signup is needed.

## Syllabus

1. **Why Local, Why Now (15 min)** – The data sovereignty case for local AI, what we're building today, and lab environment check
2. **Ollama Setup and Configuration (30 min)** – Install Ollama, pull a model, run it from the CLI, and configure it to listen on the network
3. **Model Customization (45 min)** – Modelfile labs: system prompts, parameters, and building purpose specific assistants
4. **Tailscale Mesh Networking (45 min)** – Create a free Tailscale account, install it on both VMs, authenticate, verify connectivity, and reach Ollama across the mesh
5. **Locking It Down (30 min)** – nginx reverse proxy with basic authentication in front of Ollama
6. **Wrap Up and Q&A (15 min)** – Where to go from here, hardening ideas, and open questions

## Who Should Take This Workshop

Security practitioners, sysadmins, and technical folks who want AI capability without shipping sensitive data to a third party. If you handle client data, work under confidentiality constraints, or just want to know exactly where your prompts go, this workshop is for you. It is a strong fit for:

- Red teamers and penetration testers, who get a private AI that won't leak client data during an engagement plus mesh networking they can use on assessments
- Consultants and MSSP staff who handle multiple clients' sensitive data under contract
- Anyone working in a regulated industry (healthcare, legal, finance, government) where data sovereignty is a compliance requirement, not a preference

It is also a fit for anyone who has played with a local model on one machine and wants to turn it into a real service their team can use.

## Audience Skill Level

Intermediate. You should be comfortable working in a Linux terminal and already know your way around frontier LLMs and basic AI concepts.

## Prerequisites

- Working familiarity with frontier LLMs (ChatGPT, Claude, Gemini, or similar) and basic AI/LLM concepts. You should know what a prompt is, what a model is, and what a system prompt does
- Comfort with the Linux command line (editing files, installing packages, basic troubleshooting)
- Comfort running privileged commands (sudo) to install packages and edit system config
- Basic web/HTTP literacy (the idea of clients, servers, ports, and requests)
- Basic familiarity with virtual machines (importing and running a VM in VMware)
- Helpful but not required: familiarity with networking concepts (IP addresses, ports, proxies)
- No prior experience with Ollama, Tailscale, or nginx is needed; every tool is introduced from scratch


