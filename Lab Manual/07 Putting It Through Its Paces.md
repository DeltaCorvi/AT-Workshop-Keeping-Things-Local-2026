---
author: Bronwen Aker
updated: 2026-07-16
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

# Putting It Through Its Paces

You have built the whole stack: a model on HeartOfGold, an encrypted mesh, and authentication in front of it. Now let's use it! 

This lesson reaches out to one served LLM model from Marvin in three different ways, then points it at the work that makes a private LLM worth the trouble.

## One Model, Many Harnesses

Back in [[01 What is an LLM]] you met the idea of a harness: the software wrapped around a raw model that turns it into something you can actually talk to. HeartOfGold serves exactly one model over the mesh. Everything in this lesson is a different harness in front of that same endpoint. The terminal, the TUI, and the web UI are three faces on one service, not three separate installs of the model.

> [!note] Prerequisites for This Lesson
> The three clients (curl, oterm, and Open WebUI) are already installed on Marvin. Each one talks to HeartOfGold at its tailnet address, so the mesh from [[05 Tailscale Mesh Networking]] has to be up. Confirm Marvin can reach the model before you start:
> ```shell
> curl http://heartofgold:11434/api/tags
> ```

> [!todo] Reconcile the endpoint with lesson 06 before distribution
> This lesson currently points every client at `heartofgold:11434`, the direct mesh endpoint from lesson 05. Once [[06 Locking It Down with nginx]] is finalized, the model sits behind nginx on its port with basic auth, so update the host and port here and add credentials to each client (`curl -u`, and `http://user:pass@host` for oterm and Open WebUI).

## Harness One: The Terminal

The plainest harness is a single HTTP request. From Marvin, send a one-shot prompt to the API:

```shell
curl http://heartofgold:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Explain what a reverse proxy does, in two sentences.",
  "stream": false
}'
```

You get one JSON object back with the full answer in the `response` field. Drop `"stream": false` and the API streams tokens as they generate, which is what every chat interface is doing under the hood. This is the harness you reach for in scripts: no UI, easy to pipe into `jq`, and it works anywhere `curl` does.

## Harness Two: The TUI

[oterm](https://github.com/ggozad/oterm) is a text user interface for Ollama. It runs in the terminal but gives you a real chat window with scrollback and saved conversations, no browser required. That suits Marvin, which is headless.

Point it at HeartOfGold and launch it:

```shell
export OLLAMA_HOST=http://heartofgold:11434
oterm
```

oterm reads `OLLAMA_HOST` to find the server, then lists the models HeartOfGold is serving. Pick `llama3.2`, start a chat, and hold a back-and-forth conversation. Unlike the one-shot `curl`, oterm keeps the thread, so the model sees earlier turns as context. Same model as before, different harness.

## Harness Three: The Web UI

[Open WebUI](https://openwebui.com/) is a full browser interface, the closest thing here to the hosted chat apps, except the model stays on your hardware. Run it on Marvin, pointing it at HeartOfGold:

```shell
export OLLAMA_BASE_URL=http://heartofgold:11434
open-webui serve
```

It starts a local server on port `8080` and forwards requests to HeartOfGold's Ollama over the mesh.

> [!tip] Reaching a Web UI on a Headless VM
> Marvin has no desktop, so open the interface from your host's browser instead. Point it at Marvin's address on port `8080`, for example `http://marvin:8080` over the tailnet, or Marvin's VMware IP. The first visit asks you to create a local admin account; that account lives on Marvin, not in any cloud.

Once you are in, pick `llama3.2` and chat. You now have the same model behind a polished web front end, a keyboard-driven TUI, and a raw `curl`, all reaching one endpoint on HeartOfGold.

## What Makes This Worth It

A local model earns its keep on the tasks you cannot or should not hand to a hosted one. The exercises below run in any of the three harnesses. As you try them, notice that the reason each one belongs on a local model is the same reason you built this stack.

Feed it data that can never leave. Paste in a block of logs, a vulnerability scan, or a client document and ask the model to summarize, triage, or explain it. On a hosted service that content would cross the wire to a vendor. Here it stays on hardware you control, so the sensitivity of the input stops being a blocker.

Work the way an engagement demands. For authorized red team work, a local model will draft a phishing pretext, a social-engineering scenario, or a plausible lure without the refusals and the retention that come with a hosted assistant. Nothing about the target, the client, or the tradecraft is logged to a third party.

Run it fully offline. Once the model is pulled, none of this needs internet. In an air-gapped lab or a segmented client network, the local model keeps working while a cloud API is simply unreachable.

> [!warning] Authorized Use Only
> The red team uses above assume an engagement you are authorized for, inside your rules of engagement. Local generation removes the third-party record, but it does not remove your responsibility for how the output gets used.

---

< Previous - [[06 Locking It Down with nginx]] | [[08 Wrap Up and Q&A]] - Next >
