---
author: Bronwen Aker
updated: 2026-08-12
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

[[07 Putting It All Together]] assumes Open WebUI is already sitting on Marvin, ready to run. That is how the machine is supposed to be built, but on some hosted images it never made it on. This page installs it from scratch, entirely from the terminal.

First, confirm it really is missing:

> [!marvin] Marvin · benjy
> ```shell
> which open-webui
> ```

If that prints a path, you do not need this page. Head back to [[07 Putting It All Together#Harness Two: The Web UI|07 Harness Two: The Web UI]] and carry on. If it prints nothing, read on.

## Why This Takes a Few Extra Steps

Open WebUI needs Python 3.12, and the system Python on the affected images is either too new for it or built for a microarchitecture that breaks the usual `apt`-based ways of getting an older Python. Chasing that down with `apt` or a plain `pip install` is a dead end. `uv` sidesteps the whole problem: it fetches a self-contained Python 3.12 build directly, with no dependency on whatever the system's `apt` can offer.

## Installing Open WebUI

Install `uv`:

> [!marvin] Marvin · benjy
> ```shell
> curl -LsSf https://astral.sh/uv/install.sh | sh
> source ~/.bashrc
> ```

Fetch Python 3.12 and build a virtual environment against it specifically, not the system `python3`:

> [!marvin] Marvin · benjy
> ```shell
> uv python install 3.12
> uv venv ~/.venvs/open-webui --python 3.12
> ```

Activate the environment and install Open WebUI into it:

> [!marvin] Marvin · benjy
> ```shell
> source ~/.venvs/open-webui/bin/activate
> uv pip install open-webui
> deactivate
> ```

Finally, link the venv's `open-webui` onto your normal `PATH`, so `open-webui serve` works the same way [[07 Putting It All Together]] describes, with no need to activate the venv every session:

> [!marvin] Marvin · benjy
> ```shell
> sudo ln -s ~/.venvs/open-webui/bin/open-webui /usr/local/bin/open-webui
> ```

## Verify and Return

> [!marvin] Marvin · benjy
> ```shell
> which open-webui
> ```

You should see `/usr/local/bin/open-webui`. That is everything this page needs to do. Return to [[07 Putting It All Together#Harness Two: The Web UI|07 Harness Two: The Web UI]] and pick up at "Start the service on Marvin"; from there the lesson works as written.

> [!checkpoint] Checkpoint
> You have finished this page when all of the boxes below are ticked. Work through them in order, and if one does not hold, go back to the section it came from before moving on. Tick each box as you confirm it.
>
> - [ ] Open WebUI is installed on Marvin and runnable from anywhere on the command line
> - [ ] `open-webui serve` starts cleanly, ready to pick up where 07 left off

---

> [!nav]
> [[07 Putting It All Together]]
>
> [[07 Putting It All Together#Harness Two: The Web UI|07 Harness Two: The Web UI]]
