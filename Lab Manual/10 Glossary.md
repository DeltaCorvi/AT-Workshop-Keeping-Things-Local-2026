---
author: Bronwen Aker
updated: 2026-07-23
presentation_type: Workshop
venue: Antisyphon AI Summit
---

Quick reference for the tools, protocols, and concepts used in this manual. The first time a term appears in a lesson it is introduced in place; the entries here are the canonical definition so you can jump straight to one without hunting for where it was first mentioned. Link to an entry from a lesson with `[[10 Glossary#term]]`.

### Access Control List (ACL)
In a tailnet, a policy that scopes which nodes may reach which others. Instead of every device on the mesh being able to talk to every other, an ACL lets you say, for example, that only your laptop may reach the server. It is the mesh-level version of the instinct behind putting authentication on an internal service: assume the network will be entered, and make sure entry alone does not hand over everything on it.

### Artificial Intelligence (AI)
The broad field of computer science aimed at building systems that perform tasks we normally think of as needing human intelligence, including robotics, computer vision, and natural language processing. It is a large umbrella; the [[10 Glossary#Large Language Model (LLM)|LLMs]] in this workshop sit in one corner of it.

### Base Model (Foundation Model)
The raw result of next-token prediction training, before any tuning for instructions or chat. Hand one a chunk of text and it continues in a statistically plausible way, but it has no particular urge to answer questions or follow directions. You rarely run a pure base model unless you are training your own; the models in this lab are tuned descendants of one.

### Chat Model
A model tuned to hold a multi-turn conversation, with separate system, user, and assistant roles and a template that tracks whose turn it is. It is built to carry context across many messages and to honor a [[10 Glossary#System Prompt|system prompt]]. Often tagged `-chat`.

### Context Window
How much text a model can hold in view at once, measured in [[10 Glossary#Token|tokens]] rather than words or characters. Everything in the current conversation, the system prompt included, competes for that space; once it fills, the oldest content falls out of view.

### Control Plane
The coordination layer of a mesh network, handling identity, key exchange, and access policy, deciding which devices belong and letting them find and trust each other. It is distinct from the [[10 Glossary#Data Plane|data plane]] that carries your actual traffic. Tailscale runs the control plane as a hosted service; [[10 Glossary#Headscale|Headscale]] lets you run it yourself.

### Coordination Server
The server that implements the [[10 Glossary#Control Plane|control plane]]: it authenticates devices, exchanges keys, and distributes access policy, but never sees your traffic. Tailscale hosts one for you; Headscale is a self-hosted alternative.

### Data Plane
The path your actual traffic travels, directly between devices and encrypted end to end. In a Tailscale or Headscale mesh the data plane is [[10 Glossary#WireGuard|WireGuard]] tunnels running node to node; the [[10 Glossary#Control Plane|control plane]] only helps the nodes find each other, it never carries the traffic itself.

### Defense in Depth
Layering independent security controls so that if one fails or is bypassed, the next is still standing. In this lab the encrypted mesh and the nginx authentication are two such layers: the mesh controls who can route to the service, and basic auth controls who can actually use it.

### Embedding Model
A model that does not chat at all. It turns text into vectors of numbers so software can measure how similar two pieces of text are, which is the engine behind search and [[10 Glossary#Retrieval-Augmented Generation (RAG)|RAG]].

### Exit Node
A tailnet device that routes another device's outbound internet traffic, so that traffic appears to originate from the exit node rather than the sender. On an authorized engagement, this is how you make your requests look like they come from inside the target environment.

### Generative AI
The subset of AI that produces new content rather than just classifying or predicting a label. [[10 Glossary#Large Language Model (LLM)|LLMs]] are the text branch; diffusion models are the image branch.

### Harness
The software wrapped around a raw model that turns next-token prediction into something you can use: a chat window, memory of the conversation, tool calls, a command line, an API. [[10 Glossary#Ollama|Ollama]] is a harness, and so is a web UI pointed at it. One served model can sit behind many harnesses at once.

### Headscale
An open-source, self-hosted reimplementation of Tailscale's [[10 Glossary#Coordination Server|coordination server]]. Standard Tailscale clients connect to it instead of to Tailscale's cloud, which moves the last third party, the control plane, onto hardware you own. In exchange you take on running it and keeping it alive.

### htpasswd
An Apache utility for creating and managing the username and hashed password files that HTTP basic authentication reads. On Debian and Ubuntu it ships in the `apache2-utils` package. The `-c` flag creates a new file (and overwrites an existing one), so it is used only for the first user.

### HTTP Basic Authentication
A simple authentication scheme in which the client sends a username and password with every request. The credentials are only base64 encoded, not encrypted, so basic auth depends on the transport underneath it for confidentiality. In this lab that transport is the Tailscale mesh.

### Instruct Model
A [[10 Glossary#Base Model (Foundation Model)|base model]] put through instruction tuning, extra training on examples that pair an instruction with a good response, so it handles a single standalone request well, like "summarize this" or "write a regex that matches X." It expects its input phrased as a direct instruction and is often tagged `-instruct`.

### Large Language Model (LLM)
A large file of learned numerical [[10 Glossary#Weights|weights]] that, given some input text, calculates the probability of the next [[10 Glossary#Token|token]] and repeats that one token at a time. Everything else you experience, chat, tool use, coding help, is built on top of that single prediction loop. There is no database of facts inside; associations are smeared across the weights as statistical tendencies, which is why a model can state something wrong with total confidence.

### localhost (Loopback)
The address `127.0.0.1`, reachable only from the same machine. A service bound to localhost cannot be reached from any other host, which is where Ollama sits both before you expose it on the mesh and again once nginx is fronting it.

### MagicDNS
A Tailscale feature that gives every node a name resolving to its tailnet address, so you can type `heartofgold` instead of `100.75.98.11`. Under Headscale, names resolve as a fully qualified form built from the base domain you set.

### Mesh VPN
A VPN topology in which devices connect directly to each other, peer to peer, rather than routing everything through a central concentrator. Each enrolled device can reach the others over encrypted tunnels even across different networks, which is what Tailscale and Headscale build.

### Modelfile
The recipe [[10 Glossary#Ollama|Ollama]] uses to turn an existing base model into a customized one, without retraining. Three directives do most of the work: `FROM` names the base model, `PARAMETER` sets runtime options such as [[10 Glossary#Temperature|temperature]], and `SYSTEM` holds the [[10 Glossary#System Prompt|system prompt]]. The name follows Docker's `Dockerfile` convention.

### NAT Traversal
The set of techniques that let two devices sitting behind separate routers connect directly, with nobody forwarding a port or opening the firewall to inbound traffic. Tailscale leans on this so mesh nodes reach each other with nothing listening on the perimeter for a defender to find.

### nginx
Pronounced "engine x." A high performance web server that also works as a reverse proxy. In this lab it sits in front of Ollama on HeartOfGold, adding a layer of authentication that Ollama itself does not provide.

### Ollama
A local LLM runtime, and a [[10 Glossary#Harness|harness]] in its own right. It loads a model's weights, exposes them through a local API, and gives you a CLI to pull, run, and customize models. It is what serves the model on HeartOfGold.

### Open WebUI
A full browser-based chat interface, close in feel to the hosted chat apps, that you point at a local model. In this lab it runs on Marvin and reaches Ollama on HeartOfGold over the mesh, authenticating as its own credential.

### Open Weights
Weight files that are published for download, so you can run the model on your own hardware, as with Llama, Qwen, Mistral, and Gemma. Open weights is not the same as open source: you get the finished numbers, not the training data or the code, and most ship under licenses with real restrictions. The opposite is a proprietary model such as GPT-4 or Claude, whose weights stay on the vendor's servers and reach you only through an API.

### Parameters
Another name for a model's [[10 Glossary#Weights|weights]], the internal values it learned during training. Model size is marketed as a parameter count, 7B, 13B, 33B, where B is billions; more parameters generally means more nuance captured during training and a bigger memory footprint at run time.

### Pre-Auth Key
A key minted ahead of time so a node can enroll into a tailnet non-interactively, instead of each device going through an interactive login. Handy for joining several machines at once; keep the expiration short and generate a fresh key per batch of devices.

### Reasoning Model
A model trained to work a problem step by step, out loud, before committing to an answer. If it streams its scratch work under a "Thinking..." header, it is one of these. Strong on problems that need working through, slow or distracting for something that should be a quick, short answer.

### Retrieval-Augmented Generation (RAG)
A technique that supplements a model at query time with relevant external documents, so its answers can draw on a specific corpus it was never trained on, without retraining it. It leans on [[10 Glossary#Embedding Model|embedding models]] to find the passages worth pulling in.

### Reverse Proxy
A server that receives client requests and forwards them to a back end service, then returns the service's response to the client. Because every request passes through it, a reverse proxy is a natural place to add capabilities the back end lacks, such as authentication, TLS, or request routing.

### ss
Short for socket statistics. A command line tool that lists network sockets, including which ports a machine is listening on and which processes own them. It is the modern replacement for `netstat`. The `-tlnp` switches used in this manual mean TCP sockets (`t`), listening sockets only (`l`), numeric addresses and ports without name lookups (`n`), and the owning process (`p`).

### Subnet Router
A tailnet node that advertises an entire internal subnet into the mesh, so peers can reach hosts on that subnet without a separate tunnel to each one. On an engagement it is how you reach a target's internal network as if you were sitting next to it.

### System Prompt
A standing instruction that shapes how a model behaves, its persona, tone, and constraints, set once by whoever configures the [[10 Glossary#Harness|harness]] rather than repeated by the user in every message. In a [[10 Glossary#Modelfile|Modelfile]], the `SYSTEM` block is exactly this.

### systemd
The service manager on modern Linux, responsible for starting, stopping, and supervising background services such as Ollama and nginx. An override drop-in, created with `systemctl edit`, customizes a service's unit without editing the packaged file; this lab uses one to set Ollama's `OLLAMA_HOST`.

### Tailnet
Your private Tailscale (or Headscale) network: the set of devices enrolled under your account or coordination server, each with a stable `100.x.y.z` address. Devices in the same tailnet can find and reach each other; nothing outside it can.

### Tailscale
A mesh VPN service built on [[10 Glossary#WireGuard|WireGuard]] that gets you an encrypted [[10 Glossary#Mesh VPN|mesh]] in about two minutes. It pairs a client you run on each device with a hosted [[10 Glossary#Control Plane|control plane]]; the tradeoff for the convenience is that the control plane is a third party, which [[10 Glossary#Headscale|Headscale]] removes.

### Temperature
A parameter that controls how predictable or creative a model's output is. Low temperature gives focused, consistent answers; high temperature gives more varied, unexpected ones. You set it directly in a [[10 Glossary#Modelfile|Modelfile]] with `PARAMETER temperature`.

### Token
A chunk of characters the model actually reads and generates, which might be a whole word, part of a word, or a punctuation mark. Two senses are worth keeping apart: model tokens are the linguistic units a [[10 Glossary#Context Window|context window]] is measured in, while compute tokens are what hosted services meter and bill for. Run a model locally and the per-token bill disappears.

### Training
The process of showing a model text and repeatedly adjusting its [[10 Glossary#Weights|weights]] so its next-token predictions get closer to what actually appears in the data. Base training is the expensive foundation; instruction tuning is a smaller second pass that teaches the model to follow directions. Training is where a model's knowledge and behavior get baked in, which is why you cannot simply ask it to forget something it learned.

### Weights
The learned numbers that make up a model, billions of tiny dials adjusted during [[10 Glossary#Training|training]] until the model's next-token guesses line up with reality. When training stops, those settled numbers are the model; that is literally what lands on disk when you pull one. You cannot grep them, and no single fact lives in any one place.

### WireGuard
The modern VPN protocol that Tailscale and Headscale build on, known for a small codebase and current cryptography. It provides the encrypted tunnels of the [[10 Glossary#Data Plane|data plane]]; the mesh tooling on top handles enrollment and coordination.

---

> [!navprev]
> [[09 References]]
