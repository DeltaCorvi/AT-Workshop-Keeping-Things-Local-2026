---
author: Bronwen Aker
updated: 2026-07-23
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

At the end of [[05 Tailscale Mesh Networking]] you reached HeartOfGold's model from Marvin over the encrypted mesh, and nothing was exposed to the wider network. That is real progress, but there is a gap hiding in it. Anyone, or anything, already on the [[10 Glossary#Tailnet|tailnet]] can hit [[10 Glossary#Ollama|Ollama]] with no credentials at all. This lesson closes that gap by putting nginx in front of Ollama and requiring callers to authenticate, so reaching the mesh is no longer the same as reaching the model.

## Why nginx

The mesh gives you two things: the traffic between Marvin and HeartOfGold is encrypted, and only devices you enrolled can route to each other. What it does not give you is any notion of who is calling. Once a device is on the tailnet, Ollama answers it, no questions asked. A teammate's laptop, a second container, a foothold node you added last week, all of them are anonymous and all of them are trusted equally.

That is the distinction worth sitting with: encryption is not authentication. Encryption protects the conversation from anyone outside it. Authentication decides who is allowed to start a conversation in the first place. The mesh handles the former. Nothing so far handles the latter.

[[10 Glossary#nginx|nginx]] (pronounced "engine x") is a web server: a program that listens for HTTP requests and returns responses. It can serve its own content, but it is just as commonly used as a [[10 Glossary#Reverse Proxy|reverse proxy]], standing in front of another service and forwarding requests to it. That reverse proxy role is what closes the gap here. You put nginx on HeartOfGold in front of Ollama, turn on [[10 Glossary#HTTP Basic Authentication|HTTP basic authentication]], and now every request has to carry valid credentials before nginx will pass it back to the model. A device being on the mesh is no longer enough. It has to prove who it is. That is a second, independent layer sitting behind the first, which is the whole idea behind [[10 Glossary#Defense in Depth|defense in depth]]: if one control fails or is bypassed, the next one is still standing.

## Target Architecture

This is the architecture that [[05 Tailscale Mesh Networking]] drew as its endpoint. Until now the nginx box in that diagram was a promise. In this lesson you build it.

![[lesson06_nginx_boundary.png]]

The key move is that nginx, not Ollama, is now the only thing listening on HeartOfGold's tailnet address. Ollama goes back to answering on localhost only, where nothing off the machine can reach it directly. Every request from the mesh lands on nginx first, gets checked for credentials, and only then is proxied to Ollama on `127.0.0.1`.

## Moving Ollama Back to Localhost

In lesson 05 you added a `systemd` override so Ollama would bind to HeartOfGold's tailnet address. Now that nginx is going to own that address, Ollama should go back to listening only on [[10 Glossary#localhost (Loopback)|localhost]]. Edit the override again:

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

> [!info] The Ollama CLI on HeartOfGold works normally again
> Because Ollama is back on the default localhost address, you no longer need the `OLLAMA_HOST` export you set in lesson 05 to use the `ollama` command on HeartOfGold. The CLI reaches it at `127.0.0.1:11434` on its own.

## Installing nginx

Install nginx from the distribution's package repository:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo apt update
> sudo apt install -y nginx
> ```

You will also want the `htpasswd` utility to create the credentials file. It ships in `apache2-utils`, a small collection of command line tools that come bundled with the Apache web server. Do not let the name worry you: installing this package does not install or run Apache, and it does not interfere with nginx. You are only pulling in that one utility. On Debian and Ubuntu:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo apt install -y apache2-utils
> ```

## Creating Credentials

That is the hard part done. Ollama is back on localhost, nginx is installed and running, and the `htpasswd` utility is on hand. Everything the reverse proxy needs is now in place except the one thing that gives it a reason to exist: a set of credentials to check. So before you wire nginx up as the proxy, you will create that credentials file, then point nginx at it in the next section.

Basic authentication reads usernames and hashed passwords from a file. You will create two entries in that file rather than one, because the next lesson, [[07 Putting It All Together]], reaches this service from two different clients and each one gets its own set of credentials. That costs you one extra command here and pays off in that lesson, where HeartOfGold can tell the callers apart.

> [!note] One Credential per Client
> | Credential | Client |
> | --- | --- |
> | `zaphod` | `curl`, a plain one-shot HTTP request |
> | `trillian` | Open WebUI, a chat client that runs in a browser |
>
> Basic auth gives both identical access, so this is not about restricting what any one client can do. It is about being able to tell them apart in the logs and revoke them separately.

Create the file with the first user:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo htpasswd -c /etc/nginx/.htpasswd zaphod
> ```

The `-c` flag creates the file, so use it only the first time. Add the second without it, so you append instead of overwriting:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo htpasswd /etc/nginx/.htpasswd trillian
> ```

[[10 Glossary#htpasswd|htpasswd]] prompts for a password each time and stores only its hash, never the password itself. Check your work:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo cat /etc/nginx/.htpasswd
> ```

Two lines, each a username followed by a hash. If you see only one, you used `-c` more than once and overwrote the file. Run the second command above again without it.

> [!warning] These are credentials, not Linux accounts
> `zaphod` and `trillian` exist only inside `.htpasswd`. They are not users on HeartOfGold or on Marvin, they have no home directories, and you cannot log in to anything with them. This is worth being clear about because you are still logged in to Marvin as `benjy` and to HeartOfGold as `frankie` the whole time, and those accounts have nothing to do with the credentials nginx checks.

> [!tip]- What passwords should you use?
> For this lab, pick two short, memorable passwords and write down which one goes with which username. You will type them repeatedly for the rest of the workshop, and every failed call from Marvin looks identical whether the credential is wrong or the proxy is misconfigured, which makes a forgotten password an expensive way to lose ten minutes. These VMs are disposable and live only on your own machine, so weak lab passwords cost you nothing.
>
> Nothing about that advice survives contact with production. Each of these entries is really a service account, and in production each should be long, random, generated by a password manager, and stored there rather than on a sticky note. Each should also be rotated on a schedule and revoked the moment it turns up somewhere it should not be, such as a config file in a repository or a shell history. What does carry over from the lab is the shape of the arrangement: one credential per client, so that a single compromised or retired client costs you one revocation rather than a password change for everything at once. The value of any one of them is set by what it protects. Yours here protect a toy model server on a private mesh. The next one you set might protect an inference endpoint holding customer data.

> [!info] Plain HTTP basic auth is fine here, because of the mesh
> Basic authentication sends credentials in an easily decoded form, which is why it is a bad idea over plain HTTP on an open network. On the tailnet it is a different story. Every byte between Marvin and HeartOfGold is already inside a [[10 Glossary#WireGuard|WireGuard]] tunnel, so the credentials are encrypted on the wire by the mesh itself. You are getting transport encryption from Tailscale and identity from nginx, each doing the job it is good at.

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

> [!note] What `nginx -t` Does Not Check
> The test covers syntax only. It does not check that the address in your `listen` line is one HeartOfGold actually holds, so a mistyped tailnet address still reports success here and fails at the reload instead, with a bind error that does not obviously point back at the typo. Confirm the address before you go on.

> [!hog] HeartOfGold · frankie
> ```shell
> tailscale ip -4
> grep listen /etc/nginx/sites-available/ollama
> ```

The address in the `listen` line should match the one `tailscale ip -4` prints.

When the test passes and the address checks out, reload nginx so it picks up the new site:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo systemctl reload nginx
> ```

Confirm nginx bound where you expected it to:

> [!hog] HeartOfGold · frankie
> ```shell
> ss -tlnp | grep 11434
> ```

This time you should see two lines rather than one: `127.0.0.1:11434` owned by `ollama`, and `100.x.y.z:11434` owned by `nginx`. That is the arrangement described above, visible in the output. If the nginx line is missing, check `sudo systemctl status nginx` for a bind error.

> [!tip] If nginx fails to start after a reboot
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
> curl -u zaphod http://heartofgold:11434/api/tags
> ```

curl prompts for the password, and you get the JSON list of models back, the same list you saw in lesson 05. Send a prompt the same way:

> [!marvin] Marvin · benjy
> ```shell
> curl -u zaphod http://heartofgold:11434/api/generate -d '{"model":"llama3.2","prompt":"Say hello in one sentence.","stream":false}'
> ```

> [!info] Your lesson 05 commands need `-u` now
> Every call you made in lesson 05 still works, with one change: it has to carry credentials. Anywhere you ran `curl http://heartofgold:11434/...`, you now run `curl -u zaphod http://heartofgold:11434/...`. The same applies to every other client you point at the service, which is why you created the `trillian` entry alongside `zaphod`. Both get used in [[07 Putting It All Together]].

## Why This Matters for Red Teamers

The pattern in this lesson, an authenticating reverse proxy in front of a plain internal service, is one you meet constantly on engagements, from both sides of it.

- **Defenders layer controls for a reason.** A service being on an internal or trusted network is not supposed to mean it is reachable by anyone who gets onto that network. The proxy in front of it is what turns network access into a separate question from service access. When you land on a foothold and find an internal service that just answers, that missing layer is worth noting. When you find one gated by auth, that gate is telling you the defenders thought about lateral movement.
- **Credentials become the new objective.** Once a service is behind basic auth, reaching it is no longer about routing, it is about the username and password. That shifts what is valuable during an engagement toward credential material in configs, history files, and memory, exactly the `.htpasswd` file and the shell history you just created on HeartOfGold.
- **Scoping access inside a trusted network.** Putting auth on an internal service is the same instinct as scoping a tailnet with access control lists: assume the network will be entered, and make sure entry alone does not hand over everything on it.

> [!warning] Basic auth is the floor, not the ceiling
> This lesson gives you one clean layer of identity, which is a large improvement over none. It is not the strongest option. Basic auth sends the password itself on every request rather than proving possession of it, it has no expiry, no lockout after repeated failures, and no way to give one credential less access than another. Separate entries per client get you attribution and independent revocation, but not authorization. The steps up from here are mutual TLS, where each client presents its own certificate, or a token-issuing proxy that can mint scoped credentials and expire them. Basic auth over the mesh is a solid floor to build on, not the last word.

> [!checkpoint] Checkpoint
> You have finished this lesson when all of the boxes below are ticked. Work through them in order, and if one does not hold, go back to the section it came from before moving on. Tick each box as you confirm it.
>
> - [ ] `sudo cat /etc/nginx/.htpasswd` on HeartOfGold shows two lines, `zaphod` and `trillian`, and you have both passwords written down
> - [ ] On HeartOfGold, `ss -tlnp | grep 11434` shows two listeners, `127.0.0.1:11434` owned by `ollama` and `100.x.y.z:11434` owned by `nginx`
> - [ ] From Marvin, a credential-free `curl http://heartofgold:11434/api/tags` returns `401 Unauthorized`
> - [ ] The same call with `curl -u zaphod` returns the JSON model list
> - [ ] A prompt sent with `curl -u zaphod` to `/api/generate` comes back with a `response`

---

> [!nav]
> [[05 Tailscale Mesh Networking]]
>
> [[07 Putting It All Together]]
