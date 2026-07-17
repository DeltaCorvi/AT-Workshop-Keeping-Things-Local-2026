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

# Locking It Down with nginx

At the end of [[05 Tailscale Mesh Networking]] you reached HeartOfGold's model from Marvin over the encrypted mesh, and nothing was exposed to the wider network. That is real progress, but there is a gap hiding in it. Anyone, or anything, already on the tailnet can hit Ollama with no credentials at all. This lesson closes that gap by putting nginx in front of Ollama and requiring callers to authenticate, so reaching the mesh is no longer the same as reaching the model.

## Why nginx

The mesh gives you two things: the traffic between Marvin and HeartOfGold is encrypted, and only devices you enrolled can route to each other. What it does not give you is any notion of who is calling. Once a device is on the tailnet, Ollama answers it, no questions asked. A teammate's laptop, a second container, a foothold node you added last week, all of them are anonymous and all of them are trusted equally.

That is the distinction worth sitting with: encryption is not authentication. Encryption protects the conversation from anyone outside it. Authentication decides who is allowed to start a conversation in the first place. The mesh handles the former. Nothing so far handles the latter.

nginx closes that gap. You put it on HeartOfGold as a reverse proxy in front of Ollama, turn on HTTP basic authentication, and now every request has to carry valid credentials before nginx will pass it back to the model. A device being on the mesh is no longer enough. It has to prove who it is. That is a second, independent layer sitting behind the first, which is the whole idea behind defense in depth: if one control fails or is bypassed, the next one is still standing.

## Target Architecture

This is the architecture that [[05 Tailscale Mesh Networking]] drew as its endpoint. Until now the nginx box in that diagram was a promise. In this lesson you build it.

![[lesson06_nginx_boundary.png]]

The key move is that nginx, not Ollama, is now the only thing listening on HeartOfGold's tailnet address. Ollama goes back to answering on localhost only, where nothing off the machine can reach it directly. Every request from the mesh lands on nginx first, gets checked for credentials, and only then is proxied to Ollama on `127.0.0.1`.

## Moving Ollama Back to Localhost

In lesson 05 you added a systemd override so Ollama would bind to HeartOfGold's tailnet address. Now that nginx is going to own that address, Ollama should go back to listening only on localhost. Edit the override again:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo systemctl edit ollama
> ```

Set the bind address back to localhost:

```
[Service]
Environment="OLLAMA_HOST=127.0.0.1:11434"
```

Reload and restart so the change takes effect:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo systemctl daemon-reload
> sudo systemctl restart ollama
> ```

Confirm Ollama is once again bound to localhost and nothing else:

> [!hog] HeartOfGold · frankie
> ```shell
> ss -tlnp | grep 11434
> ```

You should see `127.0.0.1:11434` and no `100.x.y.z` line. At this exact moment Marvin can no longer reach the model at all, which is expected. nginx will restore that path in a moment, this time with a lock on it.

> [!info] The Ollama CLI on HeartOfGold Works Normally Again
> Because Ollama is back on the default localhost address, you no longer need the `OLLAMA_HOST` export you set in lesson 05 to use the `ollama` command on HeartOfGold. The CLI reaches it at `127.0.0.1:11434` on its own.

## Installing nginx

Install nginx from the distribution's package repository:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo apt update
> sudo apt install -y nginx
> ```

You will also want the `htpasswd` utility to create the credentials file. On Debian and Ubuntu it ships in `apache2-utils`:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo apt install -y apache2-utils
> ```

## Creating Credentials

Basic authentication reads usernames and hashed passwords from a file. Create one with a single user. The example uses `benjy`, the account you call from on Marvin, but this name is an nginx credential and has nothing to do with any Linux account on either machine.

> [!hog] HeartOfGold · frankie
> ```shell
> sudo htpasswd -c /etc/nginx/.htpasswd benjy
> ```

The `-c` flag creates the file, so use it only the first time. To add more users later, run the same command without `-c` so you append instead of overwriting. htpasswd prompts for a password and stores only its hash, never the password itself.

> [!info] Plain HTTP Basic Auth Is Fine Here, Because of the Mesh
> Basic authentication sends credentials in an easily decoded form, which is why it is a bad idea over plain HTTP on an open network. On the tailnet it is a different story. Every byte between Marvin and HeartOfGold is already inside a WireGuard tunnel, so the credentials are encrypted on the wire by the mesh itself. You are getting transport encryption from Tailscale and identity from nginx, each doing the job it is good at.

## Configuring the Reverse Proxy

Create a site configuration for the proxy. Put it in `/etc/nginx/sites-available/ollama`:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo nano /etc/nginx/sites-available/ollama
> ```

Use the following, replacing `100.x.y.z` with HeartOfGold's tailnet address from `tailscale ip -4`:

```nginx
server {
    listen 100.x.y.z:11434;
    server_name heartofgold;

    location / {
        auth_basic "Ollama on the mesh";
        auth_basic_user_file /etc/nginx/.htpasswd;

        proxy_pass http://127.0.0.1:11434;
        proxy_set_header Host localhost:11434;
    }
}
```

Three things are worth reading closely here. nginx listens on the tailnet address on port `11434`, the same port clients used in lesson 05, so nothing about how Marvin calls the service changes except that it now needs credentials. The `auth_basic` lines turn on the credential check and point at the file you just created. And `proxy_pass` hands authenticated requests to Ollama on localhost, while `proxy_set_header Host localhost:11434` makes the request look local to Ollama, which keeps Ollama's own origin checks happy.

> [!note] Same Port, Different Listener
> nginx on `100.x.y.z:11434` and Ollama on `127.0.0.1:11434` do not collide, because they are bound to different addresses. The tailnet address and the loopback address are separate interfaces, so both can use port 11434 at the same time.

## Enabling and Testing the Config

Enable the site by linking it into `sites-enabled`, and remove the default site so it does not shadow yours:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo ln -s /etc/nginx/sites-available/ollama /etc/nginx/sites-enabled/ollama
> sudo rm /etc/nginx/sites-enabled/default
> ```

Test the configuration before you load it. nginx will tell you if anything is malformed:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo nginx -t
> ```

When the test passes, reload nginx so it picks up the new site:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo systemctl reload nginx
> ```

> [!tip] If nginx Fails to Start After a Reboot
> nginx binds to a specific tailnet address, which only exists once Tailscale is up. On the lab VMs Tailscale is already running, so this is not an issue today. On your own machines, if nginx ever fails to start on boot with an address error, it started before the tailnet interface was ready. Restarting nginx once Tailscale is up resolves it.

## Verifying from Marvin

Go back to Marvin and repeat the call from lesson 05, with no credentials:

> [!marvin] Marvin · benjy
> ```shell
> curl http://heartofgold:11434/api/tags
> ```

This time you get a `401 Unauthorized` instead of a model list. That refusal is the proof the boundary works. Marvin is still on the mesh, still able to reach HeartOfGold, and is now being turned away because it has not proven who it is.

Now make the same call with credentials, using the username and password you set with htpasswd:

> [!marvin] Marvin · benjy
> ```shell
> curl -u benjy http://heartofgold:11434/api/tags
> ```

curl prompts for the password, and you get the JSON list of models back (`llama3.2` and `qwen3.5:4b`). Send a prompt the same way:

> [!marvin] Marvin · benjy
> ```shell
> curl -u benjy http://heartofgold:11434/api/generate -d '{"model":"llama3.2","prompt":"Say hello in one sentence.","stream":false}'
> ```

> [!info] Your Lesson 05 Commands Need `-u` Now
> Every call you made in lesson 05 still works, with one change: it has to carry credentials. Anywhere you ran `curl http://heartofgold:11434/...`, you now run `curl -u benjy http://heartofgold:11434/...`. The same applies to any TUI or web client you point at the service later, each needs to be given the username and password.

## Why This Matters for Red Teamers

The pattern in this lesson, an authenticating reverse proxy in front of a plain internal service, is one you meet constantly on engagements, from both sides of it.

- **Defenders layer controls for a reason.** A service being on an internal or trusted network is not supposed to mean it is reachable by anyone who gets onto that network. The proxy in front of it is what turns network access into a separate question from service access. When you land on a foothold and find an internal service that just answers, that missing layer is worth noting. When you find one gated by auth, that gate is telling you the defenders thought about lateral movement.
- **Credentials become the new objective.** Once a service is behind basic auth, reaching it is no longer about routing, it is about the username and password. That shifts what is valuable during an engagement toward credential material in configs, history files, and memory, exactly the `.htpasswd` file and the shell history you just created on HeartOfGold.
- **Scoping access inside a trusted network.** Putting auth on an internal service is the same instinct as scoping a tailnet with access control lists: assume the network will be entered, and make sure entry alone does not hand over everything on it.

> [!warning] Basic Auth Is the Floor, Not the Ceiling
> This lesson gives you one clean layer of identity, which is a large improvement over none. It is not the strongest option. A single shared credential does not tell one caller from another, and it cannot be revoked per client without changing it for everyone. The steps up from here are mutual TLS, where each client presents its own certificate, or a token-issuing proxy that can mint and revoke per-client credentials. Basic auth over the mesh is a solid floor to build on, not the last word.

> [!checkpoint] Checkpoint
> You have finished this lesson when all of the boxes below are ticked. Work through them in order, and if one does not hold, go back to the section it came from before moving on. Tick each box as you confirm it.
>
> - [x] On HeartOfGold, `ss -tlnp | grep 11434` shows `127.0.0.1:11434` and no `100.x.y.z` line
> - [x] From Marvin, a credential-free `curl http://heartofgold:11434/api/tags` returns `401 Unauthorized`
> - [ ] The same call with `curl -u benjy` returns the JSON model list
> - [ ] A prompt sent with `curl -u benjy` to `/api/generate` comes back with a `response`

---

< Previous - [[05 Tailscale Mesh Networking]] | [[07 Putting It Through Its Paces]] - Next >
