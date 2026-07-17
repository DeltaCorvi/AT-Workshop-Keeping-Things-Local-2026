---
author: Bronwen Aker
updated: 2026-07-15
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

## What a Modelfile Does

A Modelfile is the recipe Ollama uses to turn an existing base model into a customized one. You are not retraining anything. You wrap a base model in a standing configuration: which model to start from, how it should behave, and how adventurous its output should be.

Three directives do most of the work:

- `FROM` names the base model to build on. On HeartOfGold this is `llama3.2`, which is already pulled, so building stays instant and offline.
- `PARAMETER` sets runtime options. The one you will reach for most is `temperature`, which controls how predictable or creative the output is. See [[01 What is an LLM]] for the concept.
- `SYSTEM` holds the system prompt: a standing instruction fed to the model before the conversation starts. This is where a model's persona, task, and constraints live.

> [!info]
> `FROM` must reference a model that already exists on the system. Because `llama3.2` ships pre-loaded on HeartOfGold, every example here builds on it with no download. If you point `FROM` at a model you have not pulled, Ollama will try to fetch it first.

## Example 1: A Persona Model (daffy)

The simplest useful Modelfile is a persona: a single line of identity in the `SYSTEM` block. Here is `daffy`:

```
FROM llama3.2

# set the temperature to 1 [higher is more creative, lower is more coherent]
PARAMETER temperature 1

# set the system message
SYSTEM """
You are Daffy Duck from Looney Tunes and Merrie Melodies. Answer as Daffy, only.
"""
```

Two things are doing the work. `temperature 1` keeps the model loose and playful, which suits a cartoon character. The `SYSTEM` line pins the persona so every reply stays in voice.

This model is already built on HeartOfGold, so you can run it right away:

> [!hog] HeartOfGold · frankie
> ```shell
> ollama run daffy
> >>> What's up, doc?
> ```

To build it yourself, save the Modelfile text above as `daffy.Modelfile` and run:

> [!hog] HeartOfGold · frankie
> ```shell
> ollama create daffy -f ./daffy.Modelfile
> ```

## Example 2: A Task Model (quizmaker)

Personas are the easy case. The same three directives can encode a full task, complete with steps and a required output format. `quizmaker` turns the base model into a review question generator:

```
FROM llama3.2

# set the temperature to 1 [higher is more creative, lower is more coherent]
PARAMETER temperature 1

# set the system message
SYSTEM """

# IDENTITY and PURPOSE

You are an expert on the subject defined in the input section provided below.

# GOAL

Generate questions for a student who wants to review the main concepts of the learning objectives provided in the input section provided below.

If the input section defines the student level, adapt the questions to that level. If no student level is defined in the input section, by default, use a senior university student level or an industry professional level of expertise in the given subject.

Do not answer the questions.

Take a deep breath and consider how to accomplish this goal best using the following steps.

# STEPS

- Extract the subject of the input section.

- Redefine your expertise on that given subject.

- Extract the learning objectives of the input section.

- Generate, at most, three review questions for each learning objective. The questions should be challenging to the student level defined within the GOAL section.


# OUTPUT INSTRUCTIONS

- Output in clear, human-readable Markdown.
- Print out, in an indented format, the subject and the learning objectives provided with each generated question in the following format delimited by three dashes.
Do not print the dashes. 
---
Subject: 
* Learning objective: 
    - Question 1: {generated question 1}
    - Answer 1: 

    - Question 2: {generated question 2}
    - Answer 2:
    
    - Question 3: {generated question 3}
    - Answer 3:
---


# INPUT:

INPUT:



"""
```

Notice how much more the `SYSTEM` block carries here. It defines a role, a goal, explicit steps, and a strict output format. The lines after `INPUT:` are left blank on purpose. The student supplies the subject and learning objectives at run time, and the model fills in the questions.

Like daffy, quizmaker is pre-built on HeartOfGold:

> [!hog] HeartOfGold · frankie
> ```shell
> ollama run quizmaker
> >>> Subject: Tailscale. Learning objective: explain how a mesh VPN differs from a traditional VPN.
> ```

> [!tip]
> quizmaker ships at `temperature 1`, which is fine for variety. If you want the questions to come out more consistent and tightly scoped, lower the temperature (for example `PARAMETER temperature 0.4`) and rebuild. That is the [[01 What is an LLM]] temperature idea in practice: structured tasks often want a cooler setting than freewheeling personas do.

## Build Your Own

Try modifying one of these. Change Daffy's character, or tighten quizmaker's output format, then rebuild with `ollama create` and run it. Because everything builds on the pre-loaded `llama3.2`, your edits take effect in seconds with nothing to download.

> [!checkpoint] Checkpoint
> You have finished this lesson when all of the boxes below are ticked. Work through them in order, and if one does not hold, go back to the section it came from before moving on. Tick each box as you confirm it.
>
> - [ ] On HeartOfGold, `ollama run daffy` answers in Daffy's voice
> - [ ] `ollama run quizmaker` generates review questions from a subject and learning objective
> - [ ] `ollama create <name> -f <Modelfile>` rebuilds a model from an edited Modelfile

---

< Previous - [[03 Working with Ollama]] | [[05 Tailscale Mesh Networking]] - Next >
