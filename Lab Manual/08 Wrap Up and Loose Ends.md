---
author: Bronwen Aker
updated: 2026-07-25
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

At the beginning of this workshop, you were promised control: control of the model, the network, the authentication, and the path your data takes from prompt to response. You have it now.

You built a private AI service on hardware you control. You gave it a model, connected it to a client over an encrypted mesh, put a login in front of it, and watched it tell two different callers apart. You reached that same model from a terminal and a web browser without sending the conversation to an AI provider. More importantly, you know what every layer along that path is doing and why it is there.

That is what you came here to build. Before you decide what to build next, take a minute to look at the whole thing.

## You Built It, Meshed It, and Locked It Down

You started with the model itself. Not a chatbot, an agent, or a polished web app, but a file full of learned numerical weights that predicts one token after another. [[10 Glossary#Ollama|Ollama]] loaded those weights, gave you a command line and an API, and turned the model into a service you could reach.

Then you changed its behavior. Your [[10 Glossary#Modelfile|Modelfiles]] did not retrain the model or rewrite anything inside it. They combined a base model with a [[10 Glossary#System Prompt|system prompt]] and parameters, giving you one model that played a character and another that performed a job. The same technique can make a writing assistant, a log analyst, a code reviewer, or anything else the model can support.

Next, you moved beyond one machine. [[10 Glossary#Tailscale|Tailscale]] joined HeartOfGold and Marvin in an encrypted mesh where they could find each other without opening either VM to the wider network. You bound Ollama to HeartOfGold's tailnet address, called it from Marvin, and proved that local did not have to mean trapped on one computer.

That connection created a new problem. Anything on the tailnet could reach Ollama, and Ollama had no way to ask who was calling. You moved it back to localhost and put [[10 Glossary#nginx|nginx]] in front of it. nginx took over the mesh-facing address, checked a username and password, and forwarded authenticated requests to the model. Network access and service access became two separate decisions.

Finally, you used the finished service. `curl` reached it as `zaphod`; Open WebUI reached it as `trillian`. Both were [[10 Glossary#Harness|harnesses]] wrapped around the same model, and nginx recorded which one made each request. One model, two clients, separate credentials, and a log that could tell the difference.

The complete path looks like this:

**You → harness → encrypted mesh → authenticated proxy → Ollama → model**

The response follows the same path home. Nothing in that chain is mysterious now. Each layer has one job, and you put every one of them there yourself.

## What Keeping Things Local Actually Gives You

Running a model locally does not make it private by magic. It moves the decisions.

With a hosted AI service, someone else controls the model, operates the infrastructure, changes the product, stores whatever the product stores, and decides which controls you are allowed to have. You may get excellent answers and a beautifully convenient interface, but the service remains theirs. Your prompt and attached data have to leave your control before their model can answer it.

The service you built changes that arrangement. You choose which model runs and when it changes. You decide which machines can route to it, which callers get credentials, what the interface records, how long the logs survive, and whether the system can reach the internet at all. If a client no longer belongs, you can remove one line from `.htpasswd`. If the hosted coordination plane no longer belongs in your trust boundary, you can replace Tailscale's control plane with [[10 Glossary#Headscale|Headscale]]. If the whole service needs to disappear, you can shut it down and know exactly where its data was stored.

That is the bargain at the center of self-hosting. You gain control by taking responsibility for the work a hosted provider once handled for you. You lose convenience as a price for that control.

The data does not vanish just because it stayed local, though. Open WebUI can keep chats and uploaded files on Marvin. Shell history can keep prompts typed into `curl`. nginx records who called, when, and with what client. VM snapshots and backups can preserve all of it long after the original conversation ends. Keeping things local means those copies remain yours to find, protect, retain, and delete.

It also does not mean every dependency disappeared. Unless you completed [[05b Self-Hosting the Mesh with Headscale|lesson 05b]], Tailscale still coordinates the mesh through its hosted control plane. The model weights came from somewhere. Software updates come from somewhere. Local is not a purity test. It is a boundary you can see clearly enough to decide whether it fits.

## Before This Becomes a Real Service

The system in front of you was built to make its moving parts visible in four hours. That is why its passwords were easy to type, its network held two friendly machines, and its access control fit in a small text file. Those were useful choices for a lab. They are not a production design.

> [!warning] Do Not Ship the Lab
> A working demonstration proves that the pieces fit together. It does not prove that the result is ready for sensitive data, hostile networks, unattended operation, or an audit. Start with what you built, then make decisions for the environment where it will actually live.

### Protect the Service

Keep Ollama behind the proxy and bound to localhost. The proxy is the only layer in this stack that checks identity, and bypassing it removes the lock you spent lesson 06 installing.

Basic authentication was enough to show the mechanism, but every valid credential has the same power once nginx lets it through. Ollama does not distinguish between someone asking a question and someone listing, downloading, creating, or deleting models. If different callers need different capabilities, put something in front of Ollama that can enforce authorization rather than merely admission.

For environments that need stronger identity, expiration, or scoped access, look at mutual TLS or a proxy that can validate short-lived tokens from an identity provider.

### Protect the Traffic

Basic auth sends the password with every request, along with the prompts and responses you are trying to keep private. The WireGuard tunnel encrypted that traffic between Marvin and HeartOfGold during this lab, but nginx still serves plain HTTP at the application layer. The protection belongs to the mesh path rather than to the service itself.

Add TLS in production so nginx serves HTTPS and encrypts the HTTP exchange from client to service. Its certificate also gives the client a way to verify that it reached the intended server before sending credentials or data. That protection stays with the service if it later moves off the tailnet, becomes reachable over a LAN, or sits behind another network layer. The mesh remains useful, but it is no longer the only thing keeping the conversation private.

### Protect the Network

A device joining the tailnet should not automatically earn access to everything on it. Use Tailscale's access controls to describe which users and devices may reach the model service, and review those rules as machines come and go.

Pay particular attention to [[10 Glossary#Subnet Router|subnet routers]] and [[10 Glossary#Exit Node|exit nodes]]. Both are useful because they widen what the mesh can reach, which is also why they deserve more scrutiny than an ordinary client. On an engagement, keep the model service bound to the tailnet and away from interfaces connected to the client's network.

If the hosted coordination plane is outside the boundary you want, continue with Headscale. The encrypted data path uses WireGuard either way; the question is who coordinates membership in the mesh and holds its metadata.

### Protect the Data

Decide where prompts, responses, documents, and logs are allowed to live before someone feeds the service real data. Then check the entire path, not just the model server. The friendly web interface may store far more than Ollama does, while a shell command may leave the most sensitive part of a request in history.

Set retention deliberately. Encrypt disks that may hold client or regulated data. Include VM snapshots, exported appliances, backups, and log archives in the same handling rules as the original material. A copy under your control is still a copy you are responsible for.

### Operate What You Own

Someone has to own updates for Ollama, nginx, Tailscale or Headscale, the operating system, and every harness connected to the service. Someone also has to notice when a disk fills, a certificate expires, a credential leaks, a backup fails, or a client that left six months ago can still log in.

Give every person and automated client its own identity. Rotate credentials, revoke them when their owners leave, and keep them out of source code and shell history. Decide which events deserve an alert and where the logs should go if they need to survive a compromise of HeartOfGold. Test recovery before the model service becomes important enough that losing it hurts.

These jobs are not the exciting part of running a local model. They are the part that turns a clever build into a service people can depend on.

## Loose Ends Before You Leave

If these VMs will survive beyond the workshop, take a few minutes to leave them in a state you understand:

- Save the Modelfiles, prompts, and notes you want to keep somewhere you will remember.
- Record which nginx credentials belong to which clients, preferably in a password manager rather than in the VM.
- Revoke any temporary or reusable pre-auth keys you created while enrolling machines.
- Remove HeartOfGold and Marvin from the tailnet when you no longer need them there.
- Shut the VMs down cleanly before copying, exporting, or deleting them.
- Treat an exported VM as sensitive if it contains credentials, chat history, logs, or client material.

The manual is enough to recreate the lab. Your data and credentials are the parts that require a conscious decision.

## Where to Go Next

The next useful step depends on what made you want a local model in the first place.

If you want the whole mesh under your control, build [[05b Self-Hosting the Mesh with Headscale|the Headscale track]]. It replaces Tailscale's hosted coordination service while keeping the same client and the same WireGuard data plane.

If you want better answers, try models selected for your actual workload and hardware. Compare sizes, model families, and quantizations instead of treating a larger parameter count as an automatic upgrade. `llama3.2` fit this workshop. Your code, documents, language, latency target, and available memory should choose what comes after it.

If you want the model to work with your own documents, learn about [[10 Glossary#Retrieval-Augmented Generation (RAG)|retrieval-augmented generation]]. RAG can find relevant passages in a private collection and supply them with the prompt, giving the model useful context without retraining it or sending the collection to a hosted provider.

If you want stronger identity and access control, explore Tailscale ACLs, TLS, mutual TLS, identity-aware proxies, and short-lived tokens. The basic-auth boundary you built is the floor. Now that you understand where the boundary belongs, you can replace the mechanism without rebuilding the rest of the stack.

If you want another way to use the model, add a harness. Anything that can make an HTTP request can reach the Ollama API: a script, an editor plugin, a phone app, an internal portal, or something you write yourself. The terminal and Open WebUI were examples to demonstrate possibilities, not limits.

The sources used throughout the workshop, along with more places to continue learning, are collected in [[09 References]]. If the vocabulary starts to blur after you leave the room, [[10 Glossary]] will still be here.

## One Last Look

You came into this workshop with two VMs and a question: can you run a useful AI service without handing the whole conversation to someone else?

You can. You built the model service, changed how the model behaved, carried it across an encrypted mesh, separated network access from service access, put credentials in front of it, and watched two different harnesses use it. You also saw where the simple version stops being enough and which decisions a real deployment still demands.

The point was never that everyone should self-host every model. The point was to make the trade visible. Hosted AI gives you convenience by asking you to trust someone else's model, infrastructure, policies, and boundaries. Running locally gives you the authority to choose those things yourself, then hands you the responsibility for every choice you make.

It is your model. It is your network. It is your data.

Now they can be your rules, too.

---

> [!nav]
> [[07 Putting It All Together]]
>
> [[09 References]]
