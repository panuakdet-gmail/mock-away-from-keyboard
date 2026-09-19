---
name: mock-prompt-new-session-to-work-on-the-plan
description: Write the prompt file that hands an already-written plan (a tracker folder with a spec and issues) to a fresh Fable session that will carry it out while the owner is AFK, orchestrating subagents, surviving usage-limit resets, recording usage and time, and deciding alone; then give the owner the one line to paste. Use when a plan has been discussed and written in this session and the owner wants a new session to do the work. Sibling of mock-work-all-night, which writes the plan itself from feedback notes.
disable-model-invocation: true
---

# Prompt a new session to work on the plan

The deliverable is one prompt file in the current repo plus the one line the owner pastes. You write the prompt; you do not do the work, and you do not rewrite the plan.

The plan already exists: a tracker folder (`-tracker/NAME/` in this repo's convention) with a spec, a brief of facts, and issues. The prompt points at it and adds only what the tracker cannot hold: the mandate, the AFK rules, the usage-limit machinery, the usage and time bookkeeping, the subagent mix, and the wrap-up. It must leave the new session free to reorder, merge, split or add packages when it learns something the plan did not know; the spec's problems and the owner's decisions are fixed, the path is not.

Arguments, all optional, in plain words: the tracker folder, the version or branch name, a deadline or wake time, `budget=N`, `fable=low|medium|high` (default medium). Anything missing is decided from the repo, the conversation and memory, never asked. Ask only if no plan can be found at all.

## What the owner pastes, and what the file is

Two different things, and the reply must keep them apart:

- **The prompt file** is read by the new session. The owner never pastes its contents.
- **The paste line** is one sentence the owner types into a fresh session. It names the file. It is the only thing they paste.

The paste line appears in exactly two places, each time alone inside a fenced code block with nothing else in the block: at the top of the prompt file under the heading "How to start this run", and in the reply to the owner. Say in words, right above the block, that the prompt is everything inside the box and nothing outside it.

## Settings

**Budget ceiling** (`budget=N`, 50 to 100): the highest weekly and Fable-only usage the session may reach.

- Default **95** on an ordinary day.
- Default **100** on the eve of the weekly reset: when the weekly count resets before the run is expected to end (compare the reset time from the usage command with the deadline or the estimate in the tracker). Budget that is not used before the reset is lost, so the guard can come off.
- Whatever the ceiling, the session spends only what the plan needs. The ceiling is a guard, not a target. The one exception is a plan whose last item is open-ended ("keep going, delight me"); then the ceiling is the target, and the prompt says so.

Thresholds from N: stop new packages at N minus 6, start the wrap-up at N minus 4, all writing done at N, never past N plus 1, with every figure capped at 99. If current usage is within 10 points of the ceiling, say so in the reply.

**Fable use** (`fable=`): the row for the model table.

| Level | Fable subagents do |
|---|---|
| low | Only the final review, and a bug that Opus failed to fix twice. |
| medium (default) | The final review, hard bugs with an unknown cause, a very complex package, a messy fix that spans several files, a design question Opus got wrong once. |
| high | Everything in medium, plus every package that touches core state, rendering or long-running jobs, and the review after each wave. |

## Usage and time bookkeeping

Every prompt written by this skill makes the session keep these numbers, so the owner learns what a plan of this size costs:

- At the start: the clock time, the weekly figure, the Fable-only figure and the 5-hour figure, in `status.md`.
- About every hour and before every wave: one more row with the same figures, so the log shows the cost of each wave.
- At the end, in the summary: start and end usage for weekly and Fable-only, the difference for each, the wall-clock time from start to end, and the cost of each wave taken from the log.

## Steps

1. **Read the state.** Run `date`. Run the usage command in [prompt-template.md](prompt-template.md). Read the repo's `CLAUDE.md`, the project memory notes (earlier runs, lessons, naming, commit style, branch rules), `git status` and `git branch`, the tracker's `spec.md`, `brief.md` and `status.md`, and the newest file in `-docs/prompts/`. Done when you can name the branch, the base branch, the tracker path, the test and screenshot prefixes, every project rule the session must obey, and whether the weekly reset falls inside the run.
2. **Confirm the plan is complete enough.** The spec states the problems and the owner's decisions; the brief holds the facts an agent needs (paths, environment variables, measurements); the issues each have a Problem, a Wanted list and a Done means. If a decision in the spec is still marked as proposed and the owner has confirmed it in conversation, mark it confirmed in the spec now. If something is missing, fix the tracker, do not paper over it in the prompt.
3. **Write the prompt** to `-docs/prompts/YYYY-MM-DD-VERSION-plan-prompt.md` from [prompt-template.md](prompt-template.md). Keep it short: the tracker holds the detail. Carry every project rule and every lesson from earlier runs. Compute the thresholds from the budget and paste the Fable row. Put the "How to start this run" block at the top.
4. **Check the file.** Grep it for angle brackets, em-dashes and the banned words in the owner's global `CLAUDE.md`; fix any hit.
5. **Reply to the owner** with, in this order: the file path; the sentence "Paste exactly the one line in this box into a fresh Fable session, nothing more:" followed by the paste line alone in a code block; the budget ceiling and why (ordinary day or eve of reset); the Fable level; the usage numbers now, the tracker's estimate, and when the weekly count resets; any decision made alone, with the line to edit.

The paste line is:

```
Read -docs/prompts/FILENAME in full and carry it out. I'm AFK: don't ask me anything, decide alone, and start with the setup steps.
```
