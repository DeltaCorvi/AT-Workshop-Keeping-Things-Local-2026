---
author: Bronwen Aker
updated: 2026-07-17
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

# Glossary

Quick reference for the tools, protocols, and concepts used in this manual. The first time a term appears in a lesson it is introduced in place; the entries here are the canonical definition so you can jump straight to one without hunting for where it was first mentioned. Link to an entry from a lesson with `[[10 Glossary#term]]`.

> [!todo] Glossary is a work in progress
> Seeded with the networking and nginx terms from lessons 05 and 06. Remaining tools, protocols, and concepts from 01 through 08 still need entries. Resolve before distribution.

### HTTP Basic Authentication
A simple authentication scheme in which the client sends a username and password with every request. The credentials are only base64 encoded, not encrypted, so basic auth depends on the transport underneath it for confidentiality. In this lab that transport is the Tailscale mesh.

### htpasswd
An Apache utility for creating and managing the username and hashed password files that HTTP basic authentication reads. On Debian and Ubuntu it ships in the `apache2-utils` package. The `-c` flag creates a new file (and overwrites an existing one), so it is used only for the first user.

### nginx
Pronounced "engine x." A high performance web server that also works as a reverse proxy. In this lab it sits in front of Ollama on HeartOfGold, adding a layer of authentication that Ollama itself does not provide.

### Reverse Proxy
A server that receives client requests and forwards them to a back end service, then returns the service's response to the client. Because every request passes through it, a reverse proxy is a natural place to add capabilities the back end lacks, such as authentication, TLS, or request routing.

### ss
Short for socket statistics. A command line tool that lists network sockets, including which ports a machine is listening on and which processes own them. It is the modern replacement for `netstat`. The `-tlnp` switches used in this manual mean TCP sockets (`t`), listening sockets only (`l`), numeric addresses and ports without name lookups (`n`), and the owning process (`p`).

---

> [!navprev]
> [[09 References]]
