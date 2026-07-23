---
author: Bronwen Aker
updated: 2026-07-22
presentation_type: Workshop
venue: Antisyphon AI Summit
---

Every external resource cited in this manual, gathered in one place, plus a few worth reading past the lab. Each entry notes where it comes up and why you would return to it. Entries are listed alphabetically by name.

**[7-Zip](https://www.7-zip.org/)**
The Windows extraction tool lesson 02 recommends for unpacking the large VM archive, since the built in unzipper struggles with files that size.

**[AI for Cybersecurity Professionals, with Joff Thyer and Derek Banks](https://www.antisyphontraining.com/product/ai-for-cybersecurity-professionals-with-joff-thyer-and-derek-banks/)**
The Antisyphon course to reach for when you want the data science of how LLMs actually work, past the working knowledge [[01 What is an LLM]] gives you.

**[Headscale](https://headscale.net/)**
The self-hosted coordination server from the extracurricular [[05b Self-Hosting the Mesh with Headscale]]. Source is on [GitHub](https://github.com/juanfont/headscale), and current packages are on the [releases page](https://github.com/juanfont/headscale/releases).

**[How Tailscale Works](https://tailscale.com/blog/how-tailscale-works)**
Background on the control plane, the data plane, and NAT traversal, the concepts lesson 05 leans on when it explains why nothing gets exposed to the wider network.

**[Keka](https://www.keka.io/)**
The macOS equivalent of 7-Zip, for extracting the VM archive on a Mac.

**[Midjourney](https://www.midjourney.com/)**
The image generation service mentioned in passing in lesson 01, an example of the diffusion model branch of generative AI rather than the text branch this workshop lives in.

**[nginx auth_basic module](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)**
Reference for the `auth_basic` and `auth_basic_user_file` directives you configure in [[06 Locking It Down with nginx]]. The broader [nginx documentation index](https://nginx.org/en/docs/) covers the reverse proxy directives alongside them.

**[Obsidian](https://obsidian.md/)**
The Markdown application this manual is written for and read in on Marvin. Introduced in [[00 About This Workshop]].

**[Ollama](https://ollama.com/)**
The local LLM runtime the whole lab is built on. Platform installers and the Linux install script are at [ollama.com/download](https://ollama.com/download/). Introduced in [[03 Working with Ollama]].

**[Ollama Modelfile reference](https://github.com/ollama/ollama/blob/main/docs/modelfile.md)**
Full syntax for the `FROM`, `PARAMETER`, and `SYSTEM` directives you meet in [[04 Model Customization with Modelfiles]], along with every other directive Ollama supports.

**[Open WebUI](https://openwebui.com/)**
The browser chat interface you point at HeartOfGold in [[07 Putting It All Together]], close in feel to a hosted chat app but reaching a model on your own hardware.

**[Open WebUI documentation](https://docs.openwebui.com/)**
Setup, configuration, and the details of connecting to an Ollama backend, for when you take Open WebUI past the single connection configured in the lab.

**[Tailscale](https://tailscale.com/)**
The mesh VPN that connects Marvin to HeartOfGold in [[05 Tailscale Mesh Networking]]. Create the free account the lesson needs at [login.tailscale.com/start](https://login.tailscale.com/start).

**[Tailscale access control lists](https://tailscale.com/kb/1018/acls)**
Scoping which nodes on a tailnet may reach which others, the ACL idea referenced in lessons 05 and 06 as the mesh level counterpart to putting auth on a service.

**[VMware Workstation Pro and Fusion Pro (Broadcom)](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware+Workstation+Pro&freeDownloads=true)**
The hypervisor that runs the two lab VMs, free for personal use behind a Broadcom account. Setup is covered in [[02 Setting Up Your VMs]].

**[WireGuard](https://www.wireguard.com/)**
The VPN protocol underneath both Tailscale and Headscale. The [technical whitepaper](https://www.wireguard.com/papers/wireguard.pdf) covers the protocol and its cryptography in depth if you want to understand what the mesh is actually doing on the wire.

---

> [!nav]
> [[08 Wrap Up and Loose Ends]]
>
> [[10 Glossary]]
