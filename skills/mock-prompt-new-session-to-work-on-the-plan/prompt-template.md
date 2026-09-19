# Plan prompt template

Words in CAPS are filled in. FIXED sections are copied as written, names and numbers adjusted. Example of where a finished prompt lives: `-docs/prompts/2026-09-16-v6-plan-prompt.md` in the Example App repo.

## Usage command

Reads the login token from the macOS keychain; never print the token. Paste it into the prompt so the session has it without project memory.

```sh
TOK=$(security find-generic-password -s "Claude Code-credentials" -w | python3 -c "import sys,json;print(json.load(sys.stdin)['claudeAiOauth']['accessToken'])")
curl -s -m 15 https://api.anthropic.com/api/oauth/usage -H "Authorization: Bearer $TOK" -H "anthropic-beta: oauth-2025-04-20" | python3 -c "
import sys,json,datetime
d=json.load(sys.stdin)
for k in ('five_hour','seven_day'):
    v=d[k]; r=datetime.datetime.fromisoformat(v['resets_at'].replace('Z','+00:00')).astimezone()
    print(k, v['utilization'], '% used, resets', r.strftime('%a %d %H:%M'))
for l in d.get('limits',[]):
    if l.get('scope'): print('scoped', l['scope']['model']['display_name'], l['percent'], '%')
"
```

## The prompt

---

# Prompt for PROJECT VERSION

## How to start this run

FIXED. This file is read by the new session; do not paste it. Paste exactly the one line in the box below into a fresh Fable session, nothing more:

```
Read -docs/prompts/FILENAME in full and carry it out. I'm AFK: don't ask me anything, decide alone, and start with the setup steps.
```

Everything from here to the end of the file is the instruction the session reads.

## The mission

FIXED. Carry out the plan in TRACKER. I am AFK (FILL: for how long, or until when). Do not ask me anything: decide alone, log each decision, and keep working until the plan is done or the usage ceiling below is reached. Stopping early wastes the budget I set aside for this.

FIXED. You are the orchestrator. Your own context is expensive (you run on Fable, which has its own limit), so stay lean: plan, delegate, check results, keep the tracker. Let subagents read the large files, write the code, run the tests and take the screenshots.

## The plan, and your freedom with it

FIXED. `TRACKER/spec.md` states the problems and my decisions; those are fixed. `brief.md` holds the facts. `issues/` is one suggested split into packages, written before any code; it is a starting point, not an order. Reorder, merge, split or add packages when you learn something the plan did not know, and write why in `decided-alone.md`. What must not change: the problems, the decisions, the ground rules, and "done means" for each issue.

## Setup, first 15 minutes

FIXED, names filled:

1. Run `date`. Real clock times for every stamp; never guess the time.
2. Arm the resume job before anything else: a CronCreate job every 30 minutes with the text "continue the VERSION plan: run date, check usage, read TRACKER/status.md, resume dead agents, pick up the next open item". A rate-limited session never wakes by itself; this job is what wakes it. Delete it at the wrap-up.
3. Check usage with the command below. Write the start row into the usage log in `status.md`: clock time, weekly, Fable-only, 5-hour, reset times. This row is the baseline the final report is measured against.
4. Read `CLAUDE.md`, the README, the memory notes about earlier runs, and the whole tracker.
5. Branch BRANCH from BASE (state whether it already exists). Commit nothing on any branch.
6. Keep `status.md` as the single source of truth: the usage log, one row per package with model, agent id, state, last update, next step.

## Usage and time bookkeeping

FIXED. About every hour and before every wave, add a row to the usage log: clock time, weekly, Fable-only, 5-hour, and what was running. At the end, the summary reports: usage at start and at end for weekly and Fable-only, the difference for each, the wall-clock time from the first `date` to the last, and the cost of each wave read off the log. I want to know what a plan of this size costs.

## Usage limits

FILL the snapshot line: at TIME the numbers were weekly N% (resets WHEN), Fable-only N%, 5-hour N% (resets WHEN). Re-read them yourself.

FIXED, thresholds from the budget ceiling N (see SKILL.md: 95 on an ordinary day, 100 on the eve of the weekly reset, every threshold capped at 99). FILL one sentence saying which day this is and why the ceiling is what it is.

- The plan is the goal, not the budget: stop when the plan is done and spend only what it needs. (FILL, only when the plan ends with an open-ended item: once the plan is done, keep going on that item until the ceiling.) The ceiling guards against a broken tree and no summary, which is the worst outcome.
  - At **N minus 6** on either the weekly or the Fable-only figure: stop starting new packages. Finish what is running.
  - At **N minus 4**: start the wrap-up.
  - At **N**: all writing is done. Only the cron cleanup remains. Never go past N plus 1.
  - Estimate a wave's cost from the previous one before starting it.
- **5-hour windows.** Work straight through them. When the 5-hour figure passes 85%, add a one-shot CronCreate for its reset time plus 3 minutes. If a 429 error appears, read the reset time from its text and add a one-shot CronCreate for that time plus 2 minutes. After any reset: run `date`, check usage, resume every dead subagent with SendMessage (they keep their transcript), then run the test suite before anything else, since agents may have died mid-edit.

## Subagents: which model for what

FIXED. Pass `model` on every Agent call. Parallel agents own disjoint files and re-read a file before editing it. One test suite at a time.

| Model | Use it for |
|---|---|
| **Fable** | FILL from the Fable level, naming this plan's likely hard spots. End with: the Fable-only limit is shared with the orchestrator, so keep it rare and never run two at once. |
| **Opus** | Most of the work: every package, every ordinary bug, the read-only review after each wave. |
| **Sonnet** | Easy work with a clear spec: wording, docs, screenshot runs, tests for a finished module, mechanical edits. Tell Sonnet agents to finish in one go and not end their turn while a background command runs. |

## Rules that are not negotiable

FILL from the project's `CLAUDE.md`, memory and the previous prompt: data safety, test-suite discipline, screenshot rules, house style, writing rules (no em-dashes, no YAML frontmatter, no angle brackets in markdown, American English).

## Wrap-up: when the plan is done, or at N minus 4 usage, or by DEADLINE

FIXED:

1. **Final Fable review** (one Fable subagent) of the whole diff from BASE to the working tree: bugs, regressions, dead code, wording. Fix what it finds with Opus agents, or Fable for anything hard.
2. Full test suite, alone, green, with every data environment variable set and once with none set. Put the counts in the summary.
3. `TRACKER/summary-YYYY-MM-DD.md`: what shipped in the spec's order with screenshots, what was decided alone, what was not done and why, measurements, what is unverified, what the owner should try first, the usage and time report from the bookkeeping section, a suggested commit split with messages in the project's commit style and no trailers.
4. Update the README and `CONTEXT.md` (or the project's equivalents).
5. Save a short memory note about the run: what was built, what went wrong, lessons, and the cost.
6. Delete every cron job created. Then stop.
