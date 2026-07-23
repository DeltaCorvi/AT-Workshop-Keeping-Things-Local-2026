---
author: Bronwen Aker
updated: 2026-07-22
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

# Welcome

Thank you for signing up for *Keeping Things Local: Build It, Mesh It, Lock It* at the Antisyphon AI Summit, 2026. I'm your host, Bronwen Aker, aka *Corvus*, aka *The Cybrarian*.

Large language models are powerful tools, but they come with a tradeoff: send your data to OpenAI, Anthropic, Microsoft, or Google to get results. For security practitioners, penetration testers, and anyone handling sensitive or client data, that's a problem. This workshop shows you how to build a working alternative: a private AI service running entirely on hardware you control, accessible securely across your team or engagement, with authentication built in. 

No more black boxes. No more token limits. No more sudden changes in what you can or can't do using the LLM.

You control the entire lifecycle, from user to harness to prompt 
You control how many or how few dependencies you use. 

It's your data. Shouldn't it be your rules?

# What to Expect

This workshop builds a complete service layer around a local LLM. You start with Ollama on a single VM, then expand to a two-VM setup connected over an encrypted mesh network, then lock it down with authentication. Each section builds on the previous one.

![[heartofgold_marvin_architecture.png|center]]

## Tools We Will Use

> [!hey-claude]
> organize the tools list below to reflect when things ae related, like Tailscale and Headscale are both encrypted mesh networking tools. also, doublecheck to see if any important tools have been left out by accident.


* Virtual Machines (VMs):
	* Cloud VMs via MetaCTF/Skillbit (details TBA)
	* VMware on student host (Workstation or Fusion): two Linux VMs
* Ollama: local LLM runtime, pre-installed on server VM
* `llama3.2`: pre-loaded on the server VM to avoid bandwidth issues
* Tailscale: encrypted mesh networking to connect the two VMs securely
* Headscale (extracurricular, not installed): self-hosted coordination server, for running the mesh's control plane on your own hardware after the workshop
* nginx: reverse proxy with basic authentication in front of Ollama

## Reading This Manual

This manual was written as an [Obsidian](https://obsidian.md/) vault. As such, everything is written using markdown, but if you use a reader other than Obsidian, it probably won't look the way it was intended. 

In Obsidian everything renders the way it was designed to: the colored command boxes described below, the checkpoints at the end of each lesson, and the Previous and Next links at the bottom of every page. If the full styling did not load correctly on launch, enable the bundled CSS snippets under *Settings > Appearance > CSS snippets*.

> [!NOTE] A copy of the manual is already waiting for you
> You do not have to set any of this up yourself. Obsidian is installed on the desktop VM with this manual already loaded and ready to use. There is even a handy tool to help you pull updates. You'll get the details in [[02 Setting Up Your VMs]].

### How to Read the Command Boxes

You will be working on two machines, so every command in this manual is wrapped in a colored box that names the machine it belongs to and the user you run it as.

> [!hog] HeartOfGold · frankie
> ```shell
> ollama list
> ```

> [!marvin] Marvin · benjy
> ```shell
> curl http://heartofgold:11434/api/tags
> ```

> [!bothvms] Both VMs
> ```shell
> tailscale status
> ```

The label names the machine the command **runs on**, not the window you happen to be typing into. Those are usually the same thing, but not always. 

From [[05 Tailscale Mesh Networking]] onward, you have the option of driving HeartOfGold over SSH from a terminal on Marvin, and once you do, a `HeartOfGold · frankie` command gets typed into a window sitting on Marvin. The label is still correct. It is telling you which machine will execute it.

If you are ever unsure which machine a shell belongs to, the prompt tells you: `frankie@heartofgold` or `benjy@marvin`.

## Lessons

### [[01 What is an LLM]]
This page lays out a lot of foundational information about large language models: what they actually are, where they fit in AI overall, how they are built, how they work (as far as we know), why run one locally, and the core vocabulary the rest of the workshop uses, including model types, tokens, parameters, harnesses, system prompts, and temperature.

### [[02 Setting Up Your VMs]]
Import HeartOfGold (the server) and Marvin (the client) into VMware, meet the host requirements, and log in. Ends with opening this manual in Obsidian on Marvin, desktop updater included.

### [[03 Working with Ollama]]
Confirm Ollama is running on HeartOfGold, pull and run a model from the command line, and keep a quick reference of the Ollama commands you will use throughout.

### [[04 Model Customization with Modelfiles]]
Turn the pre-loaded base model into custom ones with a Modelfile. The FROM, PARAMETER, and SYSTEM directives, a persona model (daffy) and a task model (quizmaker), and how temperature shapes output, all with no retraining.

### [[05 Tailscale Mesh Networking]]
Connect Marvin to HeartOfGold over an encrypted Tailscale mesh: create a tailnet, enroll both VMs, bind Ollama to the tailnet address, and reach the model from Marvin. Includes why the technique matters on penetration testing engagements.

### [[05b Self-Hosting the Mesh with Headscale]]
Extracurricular, for overachievers and for after the workshop. Replace Tailscale's hosted control plane with your own Headscale server, so the whole mesh runs on hardware you control. Same client and same downstream steps; you install Headscale yourself, since it is not on the lab VMs.

### [[06 Locking It Down with nginx]]
Add a login to the service. An nginx proxy sits in front of Ollama and checks a username and password, so a spot on the mesh is no longer enough to reach the model. You set up the credentials and watch an unauthenticated call get turned away.

### [[07 Putting It All Together]]
Use the finished stack. Reach the model from Marvin two ways, a terminal (curl) and a web UI (Open WebUI), and see nginx's log name each caller. Ends on what a local model is really for: sensitive data, authorized red team work, and running fully offline.

### [[08 Wrap Up and Loose Ends]]
TBD

### [[09 References]]
Every external resource cited in the manual, gathered in one place, with links to additional goodies worth exploring on your own.

### [[10 Glossary]]
A glossary of the terms and tool names used in the workshop. The first time a lesson uses a term, it links here.

---

> [!navnext]
> [[01 What is an LLM]]
