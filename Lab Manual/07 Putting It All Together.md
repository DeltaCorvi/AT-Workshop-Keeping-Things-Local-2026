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

# Putting It All Together

You have built the whole stack: a model on HeartOfGold, an encrypted mesh, and authentication in front of it. Now let's use it! 

This lesson reaches out to one served LLM model from Marvin in three different ways, then points it at the work that makes a private LLM worth the trouble.

## One Model, Many Harnesses

Back in [[01 What is an LLM]] you met the idea of a harness: the software wrapped around a raw model that turns it into something you can actually talk to. HeartOfGold serves exactly one model over the mesh. Everything in this lesson is a different harness in front of that same endpoint. The terminal, the TUI, and the web UI are three faces on one service, not three separate installs of the model.

> [!note] Prerequisites for This Lesson
> The three clients (curl, oterm, and Open WebUI) are already installed on Marvin. Each one reaches HeartOfGold at its tailnet address, so the mesh from [[05 Tailscale Mesh Networking]] has to be up. Because [[06 Locking It Down with nginx]] put nginx and basic auth in front of Ollama, every call now has to carry credentials, and the only credentials that work are the three you created with `htpasswd` in that lesson. 
> 
> * If you changed the usernames, substitute yours throughout. 
> * Have all three passwords to hand before you start. 
> * Confirm Marvin can reach the model:
> ```shell
> curl -u ford http://heartofgold:11434/api/tags
> ```
> curl prompts for the password, then returns the JSON model list. Drop the `-u ford` and you get a `401 Unauthorized` error.

> [!info] Same Endpoint, Now with Credentials
> nginx took over HeartOfGold's tailnet address on the same port `11434` that Ollama listened on in lesson 05, so the address you call does not change: it is still `heartofgold:11434`. What changed is that every client now has to authenticate, and each harness below uses its own credential from lesson 06: `ford` for curl, `zaphod` for oterm, and `trillian` for Open WebUI. curl carries them with `-u ford`. The TUI and web UI take them in the URL, as `http://<username>:<password>@heartofgold:11434`. Substitute the matching password from `htpasswd` wherever you see `<password>`.

## Harness One: The Terminal

The plainest harness is a single HTTP request. From Marvin, send a one-shot prompt to the API:

> [!marvin] Marvin · benjy
> ```shell
> curl -u ford http://heartofgold:11434/api/generate -d '{
>   "model": "llama3.2",
>   "prompt": "Explain what a reverse proxy does, in two sentences.",
>   "stream": false
> }'
> ```

You get one JSON object back with the full answer in the `response` field. Drop `"stream": false` and the API streams tokens as they generate, which is what every chat interface is doing under the hood. This is the harness you reach for in scripts: no UI, easy to pipe into `jq`, and it works anywhere `curl` does.

## Harness Two: The TUI

[oterm](https://github.com/ggozad/oterm) is a text user interface for Ollama. It runs in the terminal but gives you a real chat window with scrollback and saved conversations, no browser required. That suits Marvin, which is headless.

Point it at HeartOfGold and launch it:

> [!marvin] Marvin · benjy
> ```shell
> export OLLAMA_URL=http://zaphod:<password>@heartofgold:11434
> oterm
> ```

oterm reads `OLLAMA_URL` to find the server, and the `zaphod:<password>` prefix hands nginx the basic-auth credentials on every request. It then lists the models HeartOfGold is serving. Pick `llama3.2`, start a chat, and hold a back-and-forth conversation. Unlike the one-shot `curl`, oterm keeps the thread, so the model sees earlier turns as context. Same model as before, different harness.

## Harness Three: The Web UI

[Open WebUI](https://openwebui.com/) is a full browser interface, the closest thing here to the hosted chat apps, except the model stays on your hardware. Run it on Marvin, pointing it at HeartOfGold:

> [!marvin] Marvin · benjy
> ```shell
> export OLLAMA_BASE_URL=http://trillian:<password>@heartofgold:11434
> open-webui serve
> ```

It starts a local server on port `8080` and forwards requests to HeartOfGold's Ollama over the mesh, passing the `trillian` credentials to nginx on each one. If you would rather not keep the password in an environment variable, leave it out of `OLLAMA_BASE_URL` and set the connection instead under Open WebUI's Admin Settings, in Connections, where it takes the same URL with credentials.

> [!tip] Reaching a Web UI on a Headless VM
> Marvin has no desktop, so open the interface from your host's browser instead. Point it at Marvin's address on port `8080`, for example `http://marvin:8080` over the tailnet, or Marvin's VMware IP. The first visit asks you to create a local admin account; that account lives on Marvin, not in any cloud.

Once you are in, pick `llama3.2` and chat. You now have the same model behind a polished web front end, a keyboard-driven TUI, and a raw `curl`, all reaching one endpoint on HeartOfGold.

## Seeing All Three from the Other Side

Everything so far has been the view from Marvin. Now look at the same activity from HeartOfGold, which has been keeping a record the whole time. nginx writes every request it handles to an access log, and because each harness authenticated as a different user, that log can tell them apart.

Open a session on HeartOfGold and follow the log:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo tail -f /var/log/nginx/access.log
> ```

Leave that running, go back to Marvin, and send one request from each harness in turn. Lines appear as you go, and the second field on each is the username nginx authenticated:

```
100.x.y.z - ford [17/Aug/2026:10:14:03 +0000] "POST /api/generate HTTP/1.1" 200 1287 "-" "curl/8.5.0"
100.x.y.z - zaphod [17/Aug/2026:10:15:22 +0000] "POST /api/chat HTTP/1.1" 200 942 "-" "httpx/0.27.0"
100.x.y.z - trillian [17/Aug/2026:10:16:47 +0000] "POST /api/chat HTTP/1.1" 200 1103 "-" "python-httpx/0.27.0"
```

Three harnesses, one model, and HeartOfGold knows which is which. Try one more thing: make a call with no credentials at all, and watch what the log records instead.

> [!marvin] Marvin · benjy
> ```shell
> curl http://heartofgold:11434/api/tags
> ```

That line comes back with a `401` and a `-` where the username would be, because nginx had nobody to record. Press `Ctrl+C` to stop following the log.

> [!info] Why Separate Credentials Were Worth the Trouble
> Basic auth does not let you give `ford`, `zaphod`, and `trillian` different levels of access; all three can do everything. What separate credentials buy is everything that depends on telling callers apart. You can see which client is responsible for which traffic, and you can revoke one without disturbing the others, since deleting a single line from `.htpasswd` locks out one harness and leaves the other two working. A single shared credential gives you neither. This is the difference between authentication that identifies and authentication that merely admits.

> [!tip] What a Red Teamer Reads Here
> This log is also a record of your own activity, written by the service you are authorized to be using. On an engagement the same file is evidence: which accounts were used, from which addresses, with which tooling. Notice that the user agent field gives each client away even before you look at the username, so `curl` in a log is visible as `curl`. Defenders hunt on exactly that. It cuts both ways, and both directions are worth understanding.

## What Makes This Worth It

A local model earns its keep on the tasks you cannot or should not hand to a hosted one. The exercises below run in any of the three harnesses. As you try them, notice that the reason each one belongs on a local model is the same reason you built this stack.

Feed it data that can never leave. Paste in a block of logs, a vulnerability scan, or a client document and ask the model to summarize, triage, or explain it. On a hosted service that content would cross the wire to a vendor. Here it stays on hardware you control, so the sensitivity of the input stops being a blocker.

Work the way an engagement demands. For authorized red team work, a local model will draft a phishing pretext, a social-engineering scenario, or a plausible lure without the refusals and the retention that come with a hosted assistant. Nothing about the target, the client, or the tradecraft is logged to a third party.

Run it fully offline. Once the model is pulled, none of this needs internet. In an air-gapped lab or a segmented client network, the local model keeps working while a cloud API is simply unreachable.

> [!warning] Authorized Use Only
> The red team uses above assume an engagement you are authorized for, inside your rules of engagement. Local generation removes the third-party record, but it does not remove your responsibility for how the output gets used.

> [!checkpoint] Checkpoint
> You have finished this lesson when all of the boxes below are ticked. Work through them in order, and if one does not hold, go back to the section it came from before moving on. Tick each box as you confirm it.
>
> - [ ] From Marvin, the one-shot `curl -u ford` to `/api/generate` returns an answer
> - [ ] `oterm`, pointed at HeartOfGold as `zaphod`, connects and holds a multi-turn conversation
> - [ ] Open WebUI, running as `trillian` and reached from your host browser at `http://marvin:8080`, chats with `llama3.2`
> - [ ] `sudo tail /var/log/nginx/access.log` on HeartOfGold shows all three usernames, plus a `401` line with `-` for the unauthenticated call

---

> [!nav]
> [[06 Locking It Down with nginx]]
>
> [[08 Wrap Up and Loose Ends]]
