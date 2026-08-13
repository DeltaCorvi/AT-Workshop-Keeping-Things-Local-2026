---
author: Bronwen Aker
updated: 2026-08-11
presentation_type: Workshop
venue: Antisyphon AI Summit
---

```table-of-contents
title: # Table of Contents
minLevel: 0
maxLevel: 3
```

[[04 Model Customization with Modelfiles]] assumes that `Modelfile.daffy` and `Modelfile.quizmaker` are waiting in frankie's home directory on HeartOfGold. That is how the machine is supposed to be built, but if you find yourself on a VM where they never arrived, this page gets you back on track. You will recreate both files from the terminal alone, typing (well, pasting) them straight into the shell, with nothing to download and no file transfer involved. That last part matters: some lab environments do not allow SSH or scp in from outside, so the console session you already have is the only door, and it is all you need.

First, confirm the files really are missing:

> [!hog] HeartOfGold · frankie
> ```shell
> ls Mod*
> ```

If both files are listed, you do not need this page at all. Head back to [[04 Model Customization with Modelfiles#Example 1: A Persona Model (daffy)|04 Example 1: A Persona Model (daffy)]] and carry on. If `ls` complains that there is no such file, read on.

## How This Works

Each block below uses a [[10 Glossary#Heredoc|heredoc]], a shell feature that feeds every line between `<<'EOF'` and the closing `EOF` to a command as if you had typed them in by hand. Here the command is `cat` with its output redirected into a new file, so the shell writes everything in between to disk exactly as it appears. Quoting the delimiter as `'EOF'` is the important detail: it tells the shell to copy the text verbatim, with no variable expansion or any other interpretation along the way.

Copy each block whole, from the `cat` line through the closing `EOF` line, and paste it into your HeartOfGold session as a single paste.

**One keyboard gotcha:** `Ctrl+V` does not paste in HeartOfGold's terminal interface, because terminals reserve that combination for other uses. Paste with `Shift+Ctrl+V` instead.

While the shell is collecting the lines it shows a `>` continuation prompt instead of the usual `$`. That is the heredoc doing its job, not an error. When the shell sees the closing `EOF` it writes the file and returns you to the normal prompt.

> [!info]
> If you are working in a browser-based console and a long paste arrives garbled, press `Ctrl+C` to abandon the partial heredoc, then paste the whole block again.

## Recreate Modelfile.daffy

> [!hog] HeartOfGold · frankie
> ```shell
> cat > ~/Modelfile.daffy <<'EOF'
> FROM llama3.2
>
> # set the temperature to 1 [higher is more creative, lower is more coherent]
> PARAMETER temperature 1
>
> # set the system message
> SYSTEM """
> You are Daffy Duck from Looney Tunes and Merrie Melodies. Answer as Daffy, only.
> """
> EOF
> ```

## Recreate Modelfile.quizmaker

Same trick, longer file. The blank lines are part of the content, so paste the block exactly as it stands.

> [!hog] HeartOfGold · frankie
> ```shell
> cat > ~/Modelfile.quizmaker <<'EOF'
> FROM llama3.2
>
> # set the temperature to 1 [higher is more creative, lower is more coherent]
> PARAMETER temperature 1
> # PARAMETER temperature 0.7
>
> # set the system message
> SYSTEM """
>
> # IDENTITY and PURPOSE
>
> You are an expert on the subject defined in the input section provided below.
>
> # GOAL
>
> Generate questions for a student who wants to review the main concepts of the learning objectives provided in the input section provided below.
>
> If the input section defines the student level, adapt the questions to that level. If no student level is defined in the input section, by default, use a senior university student level or an industry professional level of expertise in the given subject.
>
> Do not answer the questions.
>
> Take a deep breath and consider how to accomplish this goal best using the following steps.
>
> # STEPS
>
> - Extract the subject of the input section.
>
> - Redefine your expertise on that given subject.
>
> - Extract the learning objectives of the input section.
>
> - Generate, at most, three review questions for each learning objective. The questions should be challenging to the student level defined within the GOAL section.
>
>
> # OUTPUT INSTRUCTIONS
>
> - Output in clear, human-readable Markdown.
> - Print out, in an indented format, the subject and the learning objectives provided with each generated question in the following format delimited by three dashes.
> Do not print the dashes.
> ---
> Subject:
> * Learning objective:
>     - Question 1: {generated question 1}
>     - Answer 1:
>
>     - Question 2: {generated question 2}
>     - Answer 2:
>
>     - Question 3: {generated question 3}
>     - Answer 3:
> ---
>
>
> # INPUT:
>
> INPUT:
>
>
>
> """
> EOF
> ```

## Verify and Return

Confirm both files landed, and that the paste came through intact:

> [!hog] HeartOfGold · frankie
> ```shell
> ls -l Mod*
> cat ~/Modelfile.daffy
> head -12 ~/Modelfile.quizmaker
> ```

`ls` should list `Modelfile.daffy` and `Modelfile.quizmaker`, with `Modelfile.daffy` the much smaller of the two. `cat` should show the whole daffy file, from `FROM llama3.2` down through the closing `"""` after Daffy's line. `head` should show `FROM llama3.2`, the temperature lines, and the start of the `# IDENTITY and PURPOSE` section for quizmaker. That is everything this page needs to confirm. Return to [[04 Model Customization with Modelfiles#Example 1: A Persona Model (daffy)|04 Example 1: A Persona Model (daffy)]] and pick up at the `ollama create daffy` step; from there the lesson works as written.

> [!checkpoint] Checkpoint
> You have finished this page when all of the boxes below are ticked. Work through them in order, and if one does not hold, go back to the section it came from before moving on. Tick each box as you confirm it.
>
> - [ ] Both Modelfiles exist in frankie's home directory again
> - [ ] Modelfile.daffy pasted through cleanly and completely, start to finish
> - [ ] Modelfile.quizmaker pasted through cleanly too, with its full `IDENTITY and PURPOSE` section intact

---

> [!nav]
> [[04 Model Customization with Modelfiles]]
>
> [[04 Model Customization with Modelfiles#Example 1: A Persona Model (daffy)|04 Example 1: A Persona Model (daffy)]]
