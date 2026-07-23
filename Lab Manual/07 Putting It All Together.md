---
author: Bronwen Aker
updated: 2026-07-22
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

You have built the whole stack: a model on HeartOfGold, an encrypted mesh, and authentication in front of it. Now let's use it! 

This lesson reaches out to one served LLM model from Marvin in two different ways, then points it at the work that makes a private LLM worth the trouble.

## One Model, Many Harnesses

Back in [[01 What is an LLM]] you met the idea of a harness: the software wrapped around a raw model that turns it into something you can actually talk to. HeartOfGold serves exactly one model over the mesh. Everything in this lesson is a different harness in front of that same endpoint. The terminal and the web UI are two faces on one service, not two separate installs of the model.

> [!note] Prerequisites for This Lesson
> Both clients (curl and [[10 Glossary#Open WebUI|Open WebUI]]) are already installed on Marvin. Each one reaches HeartOfGold at its tailnet address, so the mesh from [[05 Tailscale Mesh Networking]] has to be up. Because [[06 Locking It Down with nginx]] put nginx and basic auth in front of Ollama, every call now has to carry credentials, and the only credentials that work are the two you created with `htpasswd` in that lesson. 
> 
> * If you changed the usernames, substitute yours throughout. 
> * Have both passwords to hand before you start. 
> * Confirm Marvin can reach the model:
> ```shell
> curl -u zaphod http://heartofgold:11434/api/tags
> ```
> curl prompts for the password, then returns the JSON model list. Drop the `-u zaphod` and you get a `401 Unauthorized` error.

> [!info] Same Endpoint, Now with Credentials
> nginx took over HeartOfGold's tailnet address on the same port `11434` that Ollama listened on in lesson 05, so the address you call does not change: it is still `heartofgold:11434`. What changed is that every client now has to authenticate, and each harness below uses its own credential from lesson 06: `zaphod` for curl and `trillian` for Open WebUI. curl carries them with `-u zaphod`. The web UI takes them in the URL, as `http://<username>:<password>@heartofgold:11434`. Substitute the matching password from `htpasswd` wherever you see `<password>`.

## Harness One: The Terminal

The plainest harness is a single HTTP request. 

From Marvin, send a one-shot prompt to the API:

> [!marvin] Marvin · benjy
> ```shell
> curl -u zaphod http://heartofgold:11434/api/generate -d '{
>   "model": "llama3.2",
>   "prompt": "Explain what a reverse proxy does, in two sentences.",
>   "stream": false
> }'
> ```

You get one JSON object back with the full answer in the `response` field. Drop `"stream": false` and the API streams tokens as they generate, which is what every chat interface is doing under the hood. This is the harness you reach for in scripts: no UI, easy to pipe into `jq`, and it works anywhere `curl` does.

## Harness Two: The Web UI

[Open WebUI](https://openwebui.com/) is a full browser interface, the closest thing here to the hosted chat apps, except the model stays on your hardware. It is already installed on Marvin, so there is nothing to set up first. You start the service, create a local account, then point it across the mesh at HeartOfGold from inside the interface.

Start the service on Marvin:

> [!marvin] Marvin · benjy
> ```shell
> open-webui serve
> ```

It brings up a local server on port `8080`. On its own it looks for an Ollama running on Marvin, and there is none, because Marvin is a pure client. So it starts with no models until you give it a connection. That connection is the whole point of this harness: the model lives on HeartOfGold, and you reach it over the mesh.

Do the rest in the browser:

1. Open Firefox on Marvin's desktop and go to `http://localhost:8080`. The first visit asks you to create a local admin account, so create one now.
2. Open **Admin Settings** from your account menu, then go to **Connections**.
3. Under **Ollama**, open **Manage** (the wrench icon), and set the connection URL to `http://trillian:<password>@heartofgold:11434`, using the `trillian` password you created in [[06 Locking It Down with nginx]]. Save it.
4. Confirm the connection. Open WebUI shows a green indicator when HeartOfGold answers, which means nginx accepted the `trillian` credentials over the mesh. If it stays red, either the mesh is down or the password is wrong.
5. Open a new chat, pick `llama3.2` from the model selector, and send a prompt.

> [!tip] The Admin Account Is Local
> The account you just created lives on Marvin, not in any cloud, and it is separate from both your Linux login and the nginx credentials. It controls who may use this copy of Open WebUI, while the `trillian` credential controls what Open WebUI is allowed to ask of HeartOfGold. Two different gates.

You now have the same model behind a polished web front end and a raw `curl`, both reaching one endpoint on HeartOfGold.

> [!tip] Baking In the Connection
> You can set the connection before the interface ever loads by putting it in the environment first: run `export OLLAMA_BASE_URL=http://trillian:<password>@heartofgold:11434` on the line before `open-webui serve`. Same URL, same credentials. Setting it in the interface instead keeps the password out of your shell history.

## Seeing Both from the Other Side

Everything so far has been the view from Marvin. Now look at the same activity from HeartOfGold, which has been keeping a record the whole time. nginx writes every request it handles to an access log, and because each harness authenticated as a different user, that log can tell them apart.

Open a session on HeartOfGold and follow the log:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo tail -f /var/log/nginx/access.log
> ```

Leave that running, go back to Marvin, and send one request from each harness in turn. Lines appear as you go, and the second field on each is the username nginx authenticated:

```
100.x.y.z - zaphod [17/Aug/2026:10:14:03 +0000] "POST /api/generate HTTP/1.1" 200 1287 "-" "curl/8.5.0"
100.x.y.z - trillian [17/Aug/2026:10:16:47 +0000] "POST /api/chat HTTP/1.1" 200 1103 "-" "python-httpx/0.27.0"
```

Two harnesses, one model, and HeartOfGold knows which is which. Try one more thing: make a call with no credentials at all, and watch what the log records instead.

> [!marvin] Marvin · benjy
> ```shell
> curl http://heartofgold:11434/api/tags
> ```

That line comes back with a `401` and a `-` where the username would be, because nginx had nobody to record. Press `Ctrl+C` to stop following the log.

### Why Separate Credentials Were Worth the Trouble

Basic auth does not let you give `zaphod` and `trillian` different levels of access; both can do everything. What separate credentials buy is everything that depends on telling callers apart. You can see which client is responsible for which traffic, and you can revoke one without disturbing the other, since deleting a single line from `.htpasswd` locks out one caller and leaves the rest working. A single shared credential gives you neither. This is the difference between authentication that identifies and authentication that merely admits.

### What a Red Teamer Reads Here

This log is also a record of your own activity, written by the service you are authorized to be using. On an engagement the same file is evidence: which accounts were used, from which addresses, with which tooling. Notice that the user agent field gives each client away even before you look at the username, so `curl` in a log is visible as `curl`. Defenders hunt on exactly that. It cuts both ways, and both directions are worth understanding.

### If You Have the Time: Make the Log Talk

Two callers is enough to show the mechanism, but not enough to make the log interesting. If you finish early, or want something to take home, add more credentials and generate your own traffic.

Create a few more accounts on HeartOfGold, naming them for what they would represent in a real deployment rather than for which app you happen to be using:

> [!hog] HeartOfGold · frankie
> ```shell
> sudo htpasswd /etc/nginx/.htpasswd backup-job
> sudo htpasswd /etc/nginx/.htpasswd monitoring
> sudo htpasswd /etc/nginx/.htpasswd analyst-laptop
> ```

Then go make noise with them from Marvin. Send single prompts, send bursts, send a few in a loop with `sleep` between them, hit different endpoints, and deliberately fail some authentications by typing the wrong password. Mix `curl` in with the Open WebUI traffic you are already generating.

Now read what you produced, and see how much it gives away:

* Which callers look like a human typing, and which look like something on a timer? Regular intervals are a tell.
* Can you spot the failed logins? What does a run of `401` lines against one username look like, and how would that read to a defender at three in the morning?
* Does the user agent field agree with the story the username tells? A credential named `monitoring` making requests with a browser user agent is worth a second look.
* If you had to revoke exactly one caller right now, could you tell from this log alone which one to cut?

None of this needs new software. It is the same nginx, the same log file, and credentials you can create in one command each. The point is that identity in the log is the difference between knowing that your service was used and knowing who used it.

## What Makes This Worth It

A local model earns its keep on the tasks you cannot or should not hand to a hosted one. The exercises below run in either harness. As you try them, notice that the reason each one belongs on a local model is the same reason you built this stack.

Feed it data that can never leave. Paste in a block of logs, a vulnerability scan, or a client document and ask the model to summarize, triage, or explain it. On a hosted service that content would cross the wire to a vendor. Here it stays on hardware you control, so the sensitivity of the input stops being a blocker.

Work the way an engagement demands. For authorized red team work, a local model will draft a phishing pretext, a social-engineering scenario, or a plausible lure without the refusals and the retention that come with a hosted assistant. Nothing about the target, the client, or the tradecraft is logged to a third party.

Run it fully offline. Once the model is pulled, none of this needs internet. In an air-gapped lab or a segmented client network, the local model keeps working while a cloud API is simply unreachable.

> [!warning] Authorized Use Only
> The red team uses above assume an engagement you are authorized for, inside your rules of engagement. Local generation removes the third-party record, but it does not remove your responsibility for how the output gets used.

> [!checkpoint] Checkpoint
> You have finished this lesson when all of the boxes below are ticked. Work through them in order, and if one does not hold, go back to the section it came from before moving on. Tick each box as you confirm it.
>
> - [ ] From Marvin, the one-shot `curl -u zaphod` to `/api/generate` returns an answer
> - [ ] Open WebUI, running as `trillian` and reached from Firefox on Marvin at `http://localhost:8080`, chats with `llama3.2`
> - [ ] `sudo tail /var/log/nginx/access.log` on HeartOfGold shows both usernames, plus a `401` line with `-` for the unauthenticated call

---

> [!nav]
> [[06 Locking It Down with nginx]]
>
> [[08 Wrap Up and Loose Ends]]
