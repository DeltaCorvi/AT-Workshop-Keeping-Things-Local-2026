---
author: Bronwen Aker
updated: 2026-08-05
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

Large language models are powerful tools, but they come with a tradeoff: send your data to OpenAI, Anthropic, Microsoft, or Google to get results. For security practitioners, penetration testers, and anyone handling sensitive or client data, that's a problem. This workshop shows you how to build a working alternative: a private AI service running in an environment you control, accessible securely across your team or engagement, with authentication built in.

No more black boxes. No more token limits. No more sudden changes in what you can or can't do using the LLM.

You control the entire lifecycle, from user to harness to prompt and back again, and you choose how much of the surrounding infrastructure to operate yourself.

It's your data. Shouldn't it be your rules?

# What to Expect

This workshop builds a complete service layer around a local LLM, one piece at a time and for a reason: install the model, wrap it in a private encrypted network so only your own devices can reach it, then add a login on top so simply being on that network is not enough by itself. You start with Ollama on a single VM, then expand to a two-VM setup connected over an encrypted mesh network, then lock it down with authentication. Each section builds on the previous one.

![[heartofgold_marvin_architecture.png|center]]

## Tools We Will Use

* **Lab environment**
	* Two Ubuntu virtual machines: HeartOfGold, the model server, and Marvin, the desktop client
	* VMware Workstation or Fusion for running the VMs on your own computer
	* Cloud VMs via MetaCTF/Skillbit. Students will receive login information and other details via email.
* **Local models and customization**
	* Ollama: the local model runtime and API on HeartOfGold
	* `llama3.2`: the pre-loaded model used throughout the workshop
	* Modelfiles: Ollama recipes that combine a base model with parameters and a system prompt
* **Encrypted mesh networking**
	* Tailscale: the hosted-control-plane path used in the main lab
	* Headscale: the extracurricular, self-hosted alternative for taking control of the mesh's coordination server
* **Access and authentication**
	* nginx: the reverse proxy that sits between the mesh and Ollama
	* `htpasswd`, from `apache2-utils`: the utility used to create per-client basic-auth credentials
* **Client interfaces**
	* `curl`: the terminal client used to call Ollama's HTTP API directly
	* Open WebUI: the browser-based chat interface running on Marvin
* **Workshop manual**
	* Obsidian: the Markdown application used to read this manual and its lab-specific formatting

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

Starting in [[02 Setting Up Your VMs]], you have the option of driving HeartOfGold over SSH from a terminal on Marvin instead of typing at its console directly, using its address on the local network. Once [[05 Tailscale Mesh Networking]] is up, that same connection trades the address for a name, `heartofgold`, so there is nothing left to look up. Either way, once you do this, a `HeartOfGold · frankie` command gets typed into a window sitting on Marvin. The label is still correct. It is telling you which machine will execute it.

If you are ever unsure which machine a shell belongs to, the prompt tells you: `frankie@heartofgold` or `benjy@marvin`.

## Lessons

### [[01 What is an LLM]]
This page lays out a lot of foundational information about large language models: what they actually are, where they fit in AI overall, how they are built, how they work (as far as we know), why run one locally, and the core vocabulary the rest of the workshop uses, including model types, tokens, parameters, harnesses, system prompts, and temperature.

### [[02 Setting Up Your VMs]]
Import HeartOfGold (the server) and Marvin (the client) into VMware, meet the host requirements, and log in. Covers an early option for driving HeartOfGold over SSH instead of typing at its console, and ends with opening this manual in Obsidian on Marvin, desktop updater included.

### [[03 Working with Ollama]]
Confirm Ollama is running on HeartOfGold, pull and run a model from the command line, and keep a quick reference of the Ollama commands you will use throughout.

### [[04 Model Customization with Modelfiles]]
Turn the pre-loaded base model into custom ones with a Modelfile. The FROM, PARAMETER, and SYSTEM directives, a persona model (daffy) and a task model (quizmaker), and how temperature shapes output, all with no retraining.

### [[05 Tailscale Mesh Networking]]
So far everything has lived on one machine. This lesson reaches HeartOfGold from Marvin over [Tailscale](https://tailscale.com/), a mesh VPN that lets two devices find and reach each other over an encrypted tunnel with no firewall ports opened and nothing exposed to the internet, the same technique used to hold access to a foothold on an engagement. Create a private network of your own devices (a tailnet), enroll both VMs in it, bind Ollama to its address there, and reach the model from Marvin.

### [[05b Self-Hosting the Mesh with Headscale]]
Extracurricular, for overachievers and for after the workshop. Tailscale's mesh still depends on Tailscale's own server to coordinate which devices belong and to hand out the keys that let them trust each other. This lesson replaces that hosted piece with your own Headscale server, so the whole mesh, coordination included, runs on hardware you control. Same client and same downstream steps; you install Headscale yourself, since it is not on the lab VMs.

### [[06 Locking It Down with nginx]]
The mesh keeps traffic encrypted, but it never asks who is calling: anything on the tailnet can currently reach Ollama with no credentials at all. This lesson closes that gap. An nginx proxy sits in front of Ollama and checks a username and password, so a spot on the mesh is no longer enough to reach the model. You set up the credentials and watch an unauthenticated call get turned away.

### [[07 Putting It All Together]]
Use the finished stack. Reach the model from Marvin two ways, a terminal (curl) and a web UI (Open WebUI), and see nginx's log name each caller. Ends on what a local model is really for: sensitive data, authorized red team work, and running fully offline.

### [[08 Wrap Up and Loose Ends]]
Step back and look at the whole build: what each layer contributes, what running locally actually changes about who controls your data, and which decisions a real deployment still demands. Ends with where to take this next, and loose ends worth tying off before you leave.

### [[09 References]]
Where to go deeper on anything that felt new or half-explained along the way: the tools and sources this workshop leans on, plus further reading for continuing past the lab.

### [[10 Glossary]]
A glossary of the terms and tools used in the workshop. The first time a lesson uses a term, it links here.

---

> [!navnext]
> [[01 What is an LLM]]
