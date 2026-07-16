---
author: Bronwen Aker
updated: 2026-07-14
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```


## Setting Up Ollama 

[Ollama](https://ollama.com/) is an excellent way to get started with using and customizing local LLMs. It is available on multiple platforms, is easy to install, has extensive documentation, and is well supported. 

> [!NOTE]
> This VM already has Ollama and one or two small models installed. This information is for your reference in case you decide to install Ollama on a host or other system later. 

### Installation

Ollama is easy to install, whether you're on Mac, Windows, or Linux. 

To install Ollama on Linux, run the following command:

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

> [!info]
> When Ollama is installed on virtual machines, it is not able to access any GPUs present on your system. The good news is that you can set up Ollama on a host system easily. For more information, go to [ollama.com/download/](https://ollama.com/download/).
### Verify Ollama is Running

On HeartOfGold, the install script already enabled and started Ollama as a systemd service, so it is running by the time the install finishes. Confirm it is responding and see the pre-loaded model:

```bash
ollama list
```

To check the version:

```shell
ollama -v
```

### Pull a Model

```shell
ollama pull llama3.2
```

This command can also be used to update a local model. Only the diff will be pulled.

> [!NOTE]
> You should have at least 8 GB of RAM available to run the 7B models, 16 GB to run the 13B models, and 32 GB to run the 33B models.

> [!info] Want to try another model?
> HeartOfGold comes with `llama3.2` (3B, ~2GB) pre-loaded, so you don't need to pull anything to get started. If you want more hands-on practice with `ollama pull`, or want to compare a different model's behavior, here are a few reasonably sized options:
> - `llama3.2:1b` (1.3GB): smaller and faster than the default, good if you're short on time
> - `qwen3.5:4b` (3.4GB): newer and noticeably stronger, still a quick pull
> - `qwen3.5:9b` (6.6GB): lands in the 7B RAM tier above, a good way to feel the RAM/parameter-count relationship from [[01 What is an LLM]] in practice
>
> Pull any of these the same way: `ollama pull <model>`

## Common Ollama Commands

A quick reference for the commands you will reach for most with Ollama:

| Command                               | What it does                                                          |
| ------------------------------------- | --------------------------------------------------------------------- |
| `ollama list`                         | List installed models                                                 |
| `ollama run <model>`                  | Start an interactive chat session                                     |
| `ollama run <model> "<prompt>"`       | Send a one-shot prompt, no session                                    |
| `ollama pull <model>`                 | Download or update a model                                            |
| `ollama show <model>`                 | View a model's parameters, template, and system prompt                |
| `ollama ps`                           | Show models currently loaded in memory                                |
| `ollama stop <model>`                 | Unload a model from memory                                            |
| `ollama cp <source> <new>`            | Duplicate a model under a new name                                    |
| `ollama rm <model>`                   | Delete a model                                                        |
| `ollama create <name> -f <Modelfile>` | Build a custom model (see [[04 Model Customization with Modelfiles]]) |
| `ollama -v`                           | Print the version                                                     |

---

< Previous - [[02 Setting Up Your VMs]] | [[04 Model Customization with Modelfiles]] - Next >
