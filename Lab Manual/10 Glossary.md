---
author: Bronwen Aker
updated: 2026-07-25
presentation_type: Workshop
venue: Antisyphon AI Summit
---

Quick reference for the tools, protocols, and concepts used in this manual. The first time a term appears in a lesson it is introduced in place; the entries here are the canonical definition so you can jump straight to one without hunting for where it was first mentioned. Link to an entry from a lesson with `[[10 Glossary#term]]`.

### 401 Unauthorized
The HTTP status code a server returns when a request arrives with no valid credentials, or with the wrong ones. In this lab it is the sound of [[10 Glossary#nginx|nginx]] doing its job: a call to the model without credentials comes back 401, and the [[10 Glossary#Access Log|access log]] records a dash where the username would be. Seeing one is confirmation that the proxy is refusing anonymous callers, not evidence that something is broken.

### Access Control List (ACL)
In a tailnet, a policy that scopes which nodes may reach which others. Instead of every device on the mesh being able to talk to every other, an ACL lets you say, for example, that only your laptop may reach the server. It is the mesh-level version of the instinct behind putting authentication on an internal service: assume the network will be entered, and make sure entry alone does not hand over everything on it.

### Access Log
The file where a web server records every request it handled, one line each. [[10 Glossary#nginx|nginx]] writes its own to `/var/log/nginx/access.log`, noting the caller's address, the time, the request, the status code, the response size, and, once [[10 Glossary#Authentication|authentication]] is in play, the username it accepted. That username column is what lets one served model tell its callers apart, and it makes the log a record of your own activity as much as anyone else's.

### Air-Gapped
Describing a system with no network path to the outside world at all, isolated deliberately rather than by firewall rule. It is where a local model earns its keep most obviously: once the [[10 Glossary#Weights|weights]] are on disk, [[10 Glossary#Ollama|Ollama]] keeps answering with no internet at all, while a hosted API is simply unreachable.

### API (Application Programming Interface)
A defined set of requests a program will accept from other software, rather than from a person at a screen. Ollama exposes an HTTP API on port 11434, and that is what `curl`, [[10 Glossary#Open WebUI|Open WebUI]], and every other [[10 Glossary#Harness|harness]] in this workshop is really talking to. Because it is ordinary HTTP, anything able to make a web request can become a client, and a [[10 Glossary#Reverse Proxy|reverse proxy]] can sit in front of it without either side noticing.

### Artificial Intelligence (AI)
The broad field of computer science aimed at building systems that perform tasks we normally think of as needing human intelligence, including robotics, computer vision, and natural language processing. It is a large umbrella; the [[10 Glossary#Large Language Model (LLM)|LLMs]] in this workshop sit in one corner of it.

### Attack Surface
Every point where something could reach a service, plus everything it can do once it arrives. Each layer in this workshop widens it: the mesh gives devices a route to the model, [[10 Glossary#nginx|nginx]] adds a credential worth stealing, and a browser interface adds a second application with its own accounts. Reach and exposure grow together, which is the tradeoff behind every step of the build.

### Authentication
Establishing who a caller is. [[10 Glossary#HTTP Basic Authentication|Basic auth]] does it with a username and password; a certificate or a signed token can do it without one. It answers a different question from [[10 Glossary#Encryption|encryption]], which protects a conversation without ever asking who is on the other end, and from [[10 Glossary#Authorization|authorization]], which decides what an identified caller is allowed to do.

### Authorization
Deciding what an identified caller may do. It is the step after [[10 Glossary#Authentication|authentication]], and the one basic auth does not perform: once [[10 Glossary#nginx|nginx]] accepts a credential, every caller reaches every [[10 Glossary#Ollama|Ollama]] endpoint equally, whether that means asking a question or deleting a model. Separating them takes something that can express permissions, such as a proxy that issues scoped tokens.

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

### DERP (Designated Encrypted Relay for Packets)
Tailscale's fallback relay network, used when two nodes cannot manage a direct connection. Traffic stays encrypted end to end and the relay only forwards it, but the packets do cross somebody else's server, which matters if your reason for self-hosting was keeping everything inside your own perimeter. [[10 Glossary#Headscale|Headscale]] hands clients Tailscale's public DERP map by default, so running your own [[10 Glossary#Control Plane|control plane]] does not by itself move the relays.

### Diffusion Model
A generative model that produces images rather than text, working from noise and refining it step by step. It is the other main branch of [[10 Glossary#Generative AI|generative AI]] alongside [[10 Glossary#Large Language Model (LLM)|LLMs]], and services such as Midjourney are built on one. Nothing in this workshop runs a diffusion model; it appears here only to mark where LLMs stop.

### Embedding Model
A model that does not chat at all. It turns text into vectors of numbers so software can measure how similar two pieces of text are, which is the engine behind search and [[10 Glossary#Retrieval-Augmented Generation (RAG)|RAG]].

### Encryption
Scrambling data so that only the intended party can read it. In this lab it comes from [[10 Glossary#WireGuard|WireGuard]] underneath the mesh, which protects everything travelling between the two VMs; [[10 Glossary#TLS (Transport Layer Security)|TLS]] does the same job at the service itself, which is what you add when a service leaves the tailnet. Encryption says nothing about who is calling, and that gap is what [[10 Glossary#Authentication|authentication]] fills.

### Exit Node
A tailnet device that routes another device's outbound internet traffic, so that traffic appears to originate from the exit node rather than the sender. On an authorized engagement, this is how you make your requests look like they come from inside the target environment.

### Fine-Tuning
Continuing to train an existing model on additional data, so its [[10 Glossary#Weights|weights]] change to suit a narrower purpose. It is a real training run with real compute cost, which is what separates it from what you do in this lab: a [[10 Glossary#Modelfile|Modelfile]] leaves the weights untouched and changes only the configuration wrapped around them. [[10 Glossary#Instruction Tuning|Instruction tuning]] is one particular kind of fine-tuning.

### FQDN (Fully Qualified Domain Name)
A hostname written out in full, all the way to its domain, such as `heartofgold.magrathea.internal` instead of the bare `heartofgold`. It matters in the Headscale lab because [[10 Glossary#MagicDNS|MagicDNS]] under Tailscale resolves the short name, while a self-hosted control plane resolves names under the `base_domain` you configured. When a bare name does not resolve, the FQDN or the tailnet address will.

### Generative AI
The subset of AI that produces new content rather than just classifying or predicting a label. [[10 Glossary#Large Language Model (LLM)|LLMs]] are the text branch; diffusion models are the image branch.

### GPU (Graphics Processing Unit)
Hardware built to do many calculations in parallel, which is most of what model [[10 Glossary#Inference|inference]] consists of, and the single biggest factor in how quickly a local model answers. This lab does without one: Ollama running inside a virtual machine cannot reach the host's GPU, so everything here runs on CPU. That constraint is why the shipped model is a small one.

### Harness
The software wrapped around a raw model that turns next-token prediction into something you can use: a chat window, memory of the conversation, tool calls, a command line, an API. [[10 Glossary#Ollama|Ollama]] is a harness, and so is a web UI pointed at it. One served model can sit behind many harnesses at once.

### Headscale
An open-source, self-hosted reimplementation of Tailscale's [[10 Glossary#Coordination Server|coordination server]]. Standard Tailscale clients connect to it instead of to Tailscale's cloud, which moves the last third party, the control plane, onto hardware you own. In exchange you take on running it and keeping it alive.

### Hole Punching
A technique for getting two machines behind separate NAT devices to talk directly, by having both send outbound packets at the right moment so each one's router will accept the other's reply. It is the practical half of [[10 Glossary#NAT Traversal|NAT traversal]], and when it fails the mesh falls back to relaying through [[10 Glossary#DERP (Designated Encrypted Relay for Packets)|DERP]].

### htpasswd
An Apache utility for creating and managing the username and hashed password files that HTTP basic authentication reads. On Debian and Ubuntu it ships in the `apache2-utils` package. The `-c` flag creates a new file (and overwrites an existing one), so it is used only for the first user.

### HTTP Basic Authentication
A simple authentication scheme in which the client sends a username and password with every request. The credentials are only base64 encoded, not encrypted, so basic auth depends on the transport underneath it for confidentiality. In this lab that transport is the Tailscale mesh.

### HTTPS
HTTP carried inside a [[10 Glossary#TLS (Transport Layer Security)|TLS]] connection, so the request and response are encrypted and the client can confirm which server it reached. It is what the `s` and the padlock mean in a browser. The [[10 Glossary#nginx|nginx]] in this lab serves plain HTTP instead and leans on the mesh for [[10 Glossary#Encryption|encryption]], which is the largest single gap between this build and a production one.

### Hypervisor
The software that runs virtual machines on a physical host, handing each one virtual CPU, memory, disk, and network. VMware Workstation Pro and Fusion Pro are the hypervisors this lab is built for, and HeartOfGold and Marvin are both guests of whichever one you installed.

### Identity Provider
A service that authenticates people on another service's behalf, so the second service never handles passwords at all. [[10 Glossary#Tailscale|Tailscale]] uses one rather than running its own logins: you sign in with Google, Microsoft, GitHub, or Apple over [[10 Glossary#OAuth|OAuth]], and Tailscale trusts that provider's word about who you are. In an organization, pointing a service at the identity provider you already run is what makes central revocation possible.

### Inference
Running a trained model to get an answer, as opposed to training it in the first place. Everything you do with [[10 Glossary#Ollama|Ollama]] in this workshop is inference: the [[10 Glossary#Weights|weights]] never change, they are only loaded into memory and used. How fast it goes depends on the model's size, its [[10 Glossary#Quantization|quantization]], and whether there is a [[10 Glossary#GPU (Graphics Processing Unit)|GPU]] to run it on.

### Instruct Model
A [[10 Glossary#Base Model (Foundation Model)|base model]] put through instruction tuning, extra training on examples that pair an instruction with a good response, so it handles a single standalone request well, like "summarize this" or "write a regex that matches X." It expects its input phrased as a direct instruction and is often tagged `-instruct`.

### Instruction Tuning
An extra training pass over a [[10 Glossary#Base Model (Foundation Model)|base model]], using examples that pair an instruction with a good response, which teaches the model to follow directions rather than merely continue text. It is far smaller than the original training run, and it is what turns a base model into an [[10 Glossary#Instruct Model|instruct model]]. See [[10 Glossary#Fine-Tuning|fine-tuning]] for the general case.

### Large Language Model (LLM)
A large file of learned numerical [[10 Glossary#Weights|weights]] that, given some input text, calculates the probability of the next [[10 Glossary#Token|token]] and repeats that one token at a time. Everything else you experience, chat, tool use, coding help, is built on top of that single prediction loop. There is no database of facts inside; associations are smeared across the weights as statistical tendencies, which is why a model can state something wrong with total confidence.

### localhost (Loopback)
The address `127.0.0.1`, reachable only from the same machine. A service bound to localhost cannot be reached from any other host, which is where Ollama sits both before you expose it on the mesh and again once nginx is fronting it.

### MagicDNS
A Tailscale feature that gives every node a name resolving to its tailnet address, so you can type `heartofgold` instead of `100.75.98.11`. Under Headscale, names resolve as a fully qualified form built from the base domain you set.

### Mesh VPN
A VPN topology in which devices connect directly to each other, peer to peer, rather than routing everything through a central concentrator. Each enrolled device can reach the others over encrypted tunnels even across different networks, which is what Tailscale and Headscale build.

### Model Types
What separates one model from another is mostly what was done to it after base training, not the architecture underneath. A [[10 Glossary#Base Model (Foundation Model)|base model]] only continues text; instruction tuning makes it an [[10 Glossary#Instruct Model|instruct model]] that handles a single request, and a conversation template makes it a [[10 Glossary#Chat Model|chat model]] that can hold a thread. A [[10 Glossary#Reasoning Model|reasoning model]] works the problem out loud before answering, and an [[10 Glossary#Embedding Model|embedding model]] does not chat at all. A [[10 Glossary#Multimodal Model|multimodal model]] takes images as well as text. Nearly everything you run in this lab is an instruct or chat model.

### Modelfile
The recipe [[10 Glossary#Ollama|Ollama]] uses to turn an existing base model into a customized one, without retraining. Three directives do most of the work: `FROM` names the base model, `PARAMETER` sets runtime options such as [[10 Glossary#Temperature|temperature]], and `SYSTEM` holds the [[10 Glossary#System Prompt|system prompt]]. The name follows Docker's `Dockerfile` convention.

### Multimodal Model
A model that accepts more than one kind of input, most often text and images together. A vision model is the usual example: hand it a picture and ask a question about it. The models in this lab are text only.

### Mutual TLS (mTLS)
[[10 Glossary#TLS (Transport Layer Security)|TLS]] in which both ends present a certificate, so the server verifies the client just as the client verifies the server. The client's certificate becomes its credential, which is a real step up from a password that never expires: certificates can be issued per client, given an expiry date, and revoked centrally. It is one of the two usual upgrades from [[10 Glossary#HTTP Basic Authentication|basic auth]].

### NAT Traversal
The set of techniques that let two devices sitting behind separate routers connect directly, with nobody forwarding a port or opening the firewall to inbound traffic. Network address translation (NAT) is what makes that difficult: a router rewrites addresses on the way past, so machines behind it have no address the outside world can dial. Tailscale leans on this so mesh nodes reach each other with nothing listening on the perimeter for a defender to find.

### nginx
Pronounced "engine x." A high performance web server that also works as a reverse proxy. In this lab it sits in front of Ollama on HeartOfGold, adding a layer of authentication that Ollama itself does not provide.

### OAuth
A standard that lets one service delegate authentication to another, so you log in at a provider you already have and the first service receives a token instead of your password. It is how enrolling a device in a [[10 Glossary#Tailnet|tailnet]] works: the browser window that opens during `tailscale up` is an OAuth login at your [[10 Glossary#Identity Provider|identity provider]].

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

### Prompt
The text sent to a model for it to respond to, and the unit everything else in this workshop moves around. A prompt is more than what you typed: the [[10 Glossary#Harness|harness]] assembles it from the [[10 Glossary#System Prompt|system prompt]], the conversation so far, and your message, then hands the whole thing over as [[10 Glossary#Token|tokens]]. The reply comes back one token at a time.

### Quantization
Storing a model's [[10 Glossary#Weights|weights]] at lower numerical precision, so the file is smaller and needs less memory to run. A model published at full precision is usually distributed in several quantized versions, and the tag you pull generally encodes which one you are getting. The trade is size and speed against some loss of output quality, and it is why two downloads of nominally the same model can differ by several gigabytes.

### Reasoning Model
A model trained to work a problem step by step, out loud, before committing to an answer. If it streams its scratch work under a "Thinking..." header, it is one of these. Strong on problems that need working through, slow or distracting for something that should be a quick, short answer.

### Retrieval-Augmented Generation (RAG)
A technique that supplements a model at query time with relevant external documents, so its answers can draw on a specific corpus it was never trained on, without retraining it. It leans on [[10 Glossary#Embedding Model|embedding models]] to find the passages worth pulling in.

### Reverse Proxy
A server that receives client requests and forwards them to a back end service, then returns the service's response to the client. Because every request passes through it, a reverse proxy is a natural place to add capabilities the back end lacks, such as authentication, TLS, or request routing.

### Service Account
A credential belonging to a program rather than to a person. The [[10 Glossary#htpasswd|htpasswd]] entries in this lab are service accounts in all but name: `zaphod` is `curl`'s, `trillian` is [[10 Glossary#Open WebUI|Open WebUI]]'s, and neither is a login for a human being. Thinking of them that way is what makes one credential per client worth the trouble, since each can then be rotated or revoked without disturbing the others.

### Snapshot
A saved point-in-time state of a virtual machine that you can roll back to later. It is the quickest way to experiment safely: take one before a lesson, break whatever you like, and return to a known state in seconds. It also copies everything the VM held at that moment, credentials, chat history, and logs included, so a snapshot deserves the same handling as the data inside it.

### ss
Short for socket statistics. A command line tool that lists network sockets, including which ports a machine is listening on and which processes own them. It is the modern replacement for `netstat`. The `-tlnp` switches used in this manual mean TCP sockets (`t`), listening sockets only (`l`), numeric addresses and ports without name lookups (`n`), and the owning process (`p`).

### STUN (Session Traversal Utilities for NAT)
A protocol that lets a machine behind NAT discover what its address and port look like from the outside, which is the information two peers need before they can attempt [[10 Glossary#Hole Punching|hole punching]]. It is one of the pieces a [[10 Glossary#WireGuard|WireGuard]] mesh uses to reach a direct connection instead of relaying.

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

### TLS (Transport Layer Security)
The protocol that encrypts an HTTP connection, turning it into HTTPS, and that lets a client confirm it reached the server it intended before sending anything sensitive. This lab does without it: [[10 Glossary#nginx|nginx]] serves plain HTTP and leans on the mesh for [[10 Glossary#Encryption|encryption]]. Adding TLS is the first change to make if the service ever leaves the tailnet, since a basic auth password crossing an unencrypted link is readable by anyone in the path. Mutual TLS goes a step further and has the client present its own certificate, which turns the certificate into the credential.

### Token
A chunk of characters the model actually reads and generates, which might be a whole word, part of a word, or a punctuation mark. Two senses are worth keeping apart: model tokens are the linguistic units a [[10 Glossary#Context Window|context window]] is measured in, while compute tokens are what hosted services meter and bill for. Run a model locally and the per-token bill disappears.

### Training
The process of showing a model text and repeatedly adjusting its [[10 Glossary#Weights|weights]] so its next-token predictions get closer to what actually appears in the data. Base training is the expensive foundation; instruction tuning is a smaller second pass that teaches the model to follow directions. Training is where a model's knowledge and behavior get baked in, which is why you cannot simply ask it to forget something it learned.

### Trust Boundary
The line between what you control and what you are relying on somebody else to run correctly. Moving a model onto your own hardware moves that line rather than erasing it: the weights came from somewhere, updates come from somewhere, and unless you run [[10 Glossary#Headscale|Headscale]], the mesh's [[10 Glossary#Control Plane|control plane]] is still Tailscale's. Deciding where the line belongs for a particular piece of work is more useful than trying to remove every dependency.

### User Agent
The field in an HTTP request where a client names itself, and one of the columns in nginx's [[10 Glossary#Access Log|access log]]. `curl` announces itself as `curl`, a browser announces a browser, and a script announces whatever library it was written with. It is worth reading next to the username, because a credential named for a background job making requests with a browser user agent is a mismatch that deserves a second look.

### Weights
The learned numbers that make up a model, billions of tiny dials adjusted during [[10 Glossary#Training|training]] until the model's next-token guesses line up with reality. When training stops, those settled numbers are the model; that is literally what lands on disk when you pull one. You cannot grep them, and no single fact lives in any one place.

### WireGuard
The modern VPN protocol that Tailscale and Headscale build on, known for a small codebase and current cryptography. It provides the encrypted tunnels of the [[10 Glossary#Data Plane|data plane]]; the mesh tooling on top handles enrollment and coordination.

---

> [!navprev]
> [[09 References]]
