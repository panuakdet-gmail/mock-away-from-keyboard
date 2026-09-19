---
name: mock-work-all-night
description: Write tonight's overnight prompt file for a fresh Fable session that orchestrates subagents, spends the weekly budget up to a set ceiling (default 98%), uses Fable at a set level, resumes after limit resets, and fills spare time with extra work.
disable-model-invocation: true
---

# Work all night: write the overnight prompt

The deliverable is one prompt file in the current repo, plus a one-line paste prompt for the owner. You write the prompt; you do not do the night's work.

Arguments, all optional, in any order and in plain words: the feedback file(s), the version name, the wake time, and the two settings below. Anything missing is decided from the repo and memory or taken from the defaults, never asked. Ask only if no feedback source can be found at all.

## Settings

**Budget ceiling** (`budget=N`, default `98`): the highest weekly and Fable-only usage the night may reach. Accepts 50 to 99. The thresholds in the prompt come from it:

| Threshold | Value | Example at 98 | Example at 85 |
|---|---|---|---|
| Stop new features | N minus 6 | 92% | 79% |
| Start wrap-up | N minus 4 | 94% | 81% |
| All writing done | N | 98% | 85% |
| Never go past | N plus 1, at most 99 | 99% | 86% |

If current weekly usage is already within 10 points of the ceiling, say so in the reply: the night will be short.

**Fable use** (`fable=low`, `fable=normal`, `fable=high`, default `normal`): how freely the night session gives work to Fable subagents. Paste the matching row into the prompt's model table.

| Level | Fable subagents do | Fable-only ceiling |
|---|---|---|
| low | Only the final review, and a bug that Opus failed to fix twice. | budget ceiling minus 10 |
| normal | Hard bugs with an unknown cause, a design question Opus got wrong once, the final review. | same as the budget ceiling |
| high | Everything in normal, plus every feature that touches core state, rendering or event handling, and the review after each wave. Opus takes the rest; Sonnet stays on mechanical work. | same as the budget ceiling |

Both settings appear in the reply to the owner with the command to change them.

## Steps

1. **Read the state.** Run `date`. Run the usage command from [prompt-template.md](prompt-template.md) and note weekly, Fable-only and 5-hour figures with reset times. Read the repo's `CLAUDE.md`, the project memory notes (earlier overnight runs, lessons, naming, commit style), `git status` and `git branch`, and the newest file in `-docs/prompts/` if one exists: last night's prompt shows the project's rules and shape. Done when you can name the branch to start from, the version name, and every project rule the night must obey.
2. **Pick the version name.** Use what the owner said. If they said nothing, follow the project's naming memory (a follow-up on an open branch is a pass of that version, not a new number).
3. **Harvest every request.** Read each feedback source in full, including embedded screenshots. A request is any note, arrow line, answer or free bullet the owner wrote. Sort them into: bugs and layout faults; changes asked for; items left blank that need judgement; choices to leave alone and carry into the morning questions. Done when every owner-written line maps to exactly one of those groups.
4. **Write the prompt** to `-docs/prompts/YYYY-MM-DD-VERSION-overnight-prompt.md`, following [prompt-template.md](prompt-template.md), with the thresholds and the Fable row computed from the settings. Write each request as a concrete, self-contained bullet (what is wrong, what done looks like, which screenshot shows it), so the night session never has to reinterpret the owner's shorthand. Carry every project rule from step 1 and every lesson from earlier nights.
5. **Check the file.** Grep it for angle brackets, em-dashes and banned words from the owner's global `CLAUDE.md`; fix any hit.
6. **Reply to the owner** with: the file path; the paste prompt in a code block; the budget ceiling and Fable level used; the usage numbers and what they mean for tonight (how much budget is left, when the weekly count resets relative to the wake time); any decision you made alone that they may want to change, with the line to edit.

The paste prompt is:

```
Read -docs/prompts/FILENAME in full and carry it out tonight. I'm asleep until morning: don't ask me anything, decide alone, and start with the setup steps.
```
