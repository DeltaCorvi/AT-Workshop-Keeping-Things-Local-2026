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

# Tailscale Mesh Networking

Up to now everything has lived on a single VM. HeartOfGold runs Ollama, and you talk to it from HeartOfGold itself. The point of this workshop is a service you can actually reach from somewhere else, securely, so this lesson connects a second machine, Marvin, to HeartOfGold over an encrypted mesh network built with [Tailscale](https://tailscale.com/).

## Why Tailscale

You have a private LLM on HeartOfGold. You want to use it from Marvin, and later from your laptop, your phone, or a teammate's box, without doing any of the things that usually make a service reachable: no opening firewall ports, no port forwarding on a router, no exposing Ollama to the local network or the public internet. Every one of those options widens your attack surface, which is exactly what this workshop is trying to avoid.

Tailscale solves this with a mesh VPN built on the WireGuard protocol. Each device you enroll joins a private network called a tailnet and gets a stable address in the `100.x.y.z` range. Devices then talk directly to each other through encrypted tunnels, even when they sit on different networks behind different routers. Nothing is published to the outside world; the two machines simply find each other and connect.

> [!info] Control Plane vs Data Plane
> Tailscale runs a coordination server that handles identity, key exchange, and access policy. That is the control plane. Your actual traffic, the data plane, flows directly between your devices and is encrypted end to end. Tailscale's servers help your machines find each other, but your private traffic never passes through them.

> [!tip] The Easy Button, and the Fully Local Alternative
> We use Tailscale here because it gets you a working mesh in about two minutes, and because it earns its place on engagements. The tradeoff for this workshop is that the *control plane* is Tailscale's hosted server, a third party. When you need that control plane inside your own perimeter too, for an internal network or a stricter trust boundary, you self-host it with Headscale and keep the same client and the same steps. See [[05b Self-Hosting the Mesh with Headscale]].

## Target Architecture

This is the final architecture once the mesh is up and nginx (lesson 06) is in place. Before nginx exists, Marvin's two interfaces talk to Ollama directly instead of going through the basic auth layer.

![[heartofgold_marvin_architecture.png]]

## Why Tailscale Matters for Red Teamers

The same properties that make Tailscale convenient here make it useful on an engagement. This is why the workshop spends time on it rather than just hardcoding an IP address.

- **Persistent, encrypted access.** A Tailscale node on a foothold gives you durable access back to that host. The tunnel is encrypted, survives the host changing networks or IP, and traverses NAT without any inbound ports, so there is nothing listening on the perimeter for a defender to find.
- **Reaching internal services.** A subnet router advertises an entire internal subnet into your tailnet. From your own box you can then reach hosts on the target's internal network as if you were sitting next to them, without standing up a separate tunnel for every host.
- **Pivoting with exit nodes.** An exit node routes your traffic out through the foothold, so your requests appear to originate from inside the target environment.
- **Clean, scoped access.** MagicDNS gives every node a name instead of an IP, and access control lists let you scope exactly which nodes can reach which, so a shared tailnet stays controlled rather than wide open.

> [!warning] Authorization and Opsec
> Use this only on engagements you are authorized for, and stay inside your rules of engagement. Remember that Tailscale's coordination server is a third party that sees device metadata such as names, keys, and connection times, even though it never sees your traffic. Factor that into your opsec. When that metadata exposure is unacceptable, self-host the control plane with [[05b Self-Hosting the Mesh with Headscale]].

## Creating a Tailscale Account

Before a device can join a tailnet, there has to be a tailnet for it to join, and that is what your account is. Tailscale's coordination server uses your identity to decide which devices belong to your private network and to hand out the keys that let those devices find and trust each other. No account means no tailnet, which is why registration comes first.

Tailscale does not run its own username and password system. It hands authentication off to an identity provider you already use, such as Google, Microsoft, GitHub, or Apple, over standard OAuth. You pick a provider, log in there, and Tailscale trusts that provider to vouch for who you are.

> [!info] The Free Plan is Enough
> Signing up with a personal email account, for example a `@gmail.com` address, puts you on Tailscale's free Personal plan, which covers everything in this lab, including MagicDNS, exit nodes, and access control lists. It is a real free tier, not a trial.

To register, go to [login.tailscale.com/start](https://login.tailscale.com/start), choose your identity provider, and authenticate. That creates your tailnet and drops you into the admin console at [login.tailscale.com](https://login.tailscale.com/), where every device you enroll will show up.

## Installing Tailscale

> [!note]
> Tailscale is already installed on HeartOfGold and Marvin, so you can skip this step on the lab VMs. It is here so you can set Tailscale up on your own machines later.

On Linux, one script installs it:

```shell
curl -fsSL https://tailscale.com/install.sh | sh
```

Tailscale also has installers for macOS, Windows, iOS, and Android, available at [tailscale.com/download](https://tailscale.com/download/).

## Joining the Mesh

On each VM, bring Tailscale up and authenticate it into your tailnet:

> [!bothvms] Both VMs
> ```shell
> sudo tailscale up
> ```

The command prints a URL. Open it in a browser, log in with the account you just created, and that device joins your tailnet. Run this on both HeartOfGold and Marvin, logging into the same account each time so they land in the same tailnet.

Confirm both machines are connected:

> [!bothvms] Both VMs
> ```shell
> tailscale status
> ```

You will see each device with its name and `100.x.y.z` address. The same list appears in the admin console. Because MagicDNS is on by default, each machine also picks up a name, so `heartofgold` and `marvin` resolve to their tailnet addresses without you memorizing the numbers.

> [!tip] Use the Names, Not the Numbers
> Anywhere you would type a tailnet IP, you can use the MagicDNS name instead. `heartofgold` is easier to remember and read than `100.75.98.11`.

> [!tip] Hot Tip: Drive HeartOfGold from Marvin over SSH
> Now that both machines are on the tailnet, you no longer have to sit at HeartOfGold's headless console for the rest of this lab. From Marvin, open a terminal and connect:
> ```shell
> ssh frankie@heartofgold
> ```
> Every HeartOfGold command from here on can run inside that SSH session, and its output lands in Marvin's terminal, where you can select and copy it like any other text. That sidesteps the headless copy/paste problem entirely, including grabbing the tailnet IP in the next step. HeartOfGold ships with its SSH server running, so this should connect straight away. If it is refused, confirm the service is up with `sudo systemctl status ssh`, and if it is missing entirely, `sudo apt install -y openssh-server` then `sudo systemctl enable --now ssh`.
>
> Two terminal windows side by side works well: one logged in to Marvin, one holding the SSH session to HeartOfGold. With the manual open on Marvin, you can copy a command straight into whichever window it belongs to.
>
> Once you do this, both windows are on your Marvin desktop, so watch the command boxes rather than the window. A `HeartOfGold · frankie` box means the command runs on HeartOfGold, which now means the SSH window. The shell prompt is the tiebreaker: `frankie@heartofgold` or `benjy@marvin`.

## Exposing Ollama on the Mesh

By default Ollama listens only on `127.0.0.1`, so nothing off the machine can reach it, including Marvin. You will bind it to HeartOfGold's tailnet address instead, so the model answers only over the encrypted mesh and not on the VM's other interfaces.

First, find HeartOfGold's tailnet address:

> [!hog] HeartOfGold · frankie
> ```shell
> tailscale ip -4
> ```

> [!tip] Write This Address Down Now!
> The command you just typed is going to return an IP address, but because HeartOfGold is headless, there is no easy copy/paste from its console. Capture the IP address before you continue: Jot the four octets on paper, or take a screenshot of the hypervisor console window on your host so you can read the address back while you type.

Then add a systemd override so Ollama binds to that address. Run:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo systemctl edit ollama
> ```

and add the following, replacing the address with the one `tailscale ip -4` gave you:

```
[Service]
Environment="OLLAMA_HOST=100.x.y.z:11434"
```

Reload and restart so the change takes effect:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo systemctl daemon-reload
> sudo systemctl restart ollama
> ```

Confirm Ollama is now listening on the tailnet address rather than localhost:

> [!hog] HeartOfGold · frankie
> ```shell
> ss -tlnp | grep 11434
> ```

You should see it bound to your `100.x.y.z` address on port `11434`, not `127.0.0.1`.

> [!info] Using the Ollama CLI on the Server after This
> Once Ollama binds to the tailnet address, its own command line on HeartOfGold can no longer reach it at the default localhost. If you want to run `ollama` on HeartOfGold itself, point it at the tailnet address first:
> ```shell
> export OLLAMA_HOST=$(tailscale ip -4):11434
> ```

> [!note] Minimal Exposure by Design
> Binding to the tailnet address means the model responds only to devices on your encrypted mesh, and nothing else. [[06 Locking It Down with nginx]] takes the next step: moving Ollama back to localhost and standing up nginx on the mesh with authentication in front of it, so even mesh peers have to prove who they are.

## Verifying the Connection

Everything is wired up. From Marvin, reach HeartOfGold's Ollama across the mesh:

> [!marvin] Marvin · benjy
> ```shell
> curl http://heartofgold:11434/api/tags
> ```

You should get back a JSON list of the models installed on HeartOfGold, which is `llama3.2` unless you pulled others in lesson 03. That response is the proof: Marvin reached the model on HeartOfGold over the encrypted tailnet, with nothing exposed to the wider network.

To watch it actually generate, send a prompt:

> [!marvin] Marvin · benjy
> ```shell
> curl http://heartofgold:11434/api/generate -d '{"model":"llama3.2","prompt":"Say hello in one sentence.","stream":false}'
> ```

> [!checkpoint] Checkpoint
> You have finished this lesson when all of the boxes below are ticked. Work through them in order, and if one does not hold, go back to the section it came from before moving on. Tick each box as you confirm it.
>
> - [ ] `tailscale status` on both VMs lists HeartOfGold and Marvin with `100.x.y.z` addresses
> - [ ] The MagicDNS name `heartofgold` resolves from Marvin
> - [ ] On HeartOfGold, `ss -tlnp | grep 11434` shows Ollama bound to the tailnet address
> - [ ] From Marvin, `curl http://heartofgold:11434/api/tags` returns the JSON model list

---

> [!nav]
> [[04 Model Customization with Modelfiles]]
>
> [[06 Locking It Down with nginx]]
