# Overnight prompt template

Words in CAPS are filled in. Sections marked FIXED are copied as written (adjust only names and numbers). Sections marked FILL come from the harvest. Example of where a finished prompt lives: `-docs/prompts/2026-09-14-v4-overnight-prompt.md` in the Example App repo (`~/projects/example-app`).

## Usage command

Reads the login token from the macOS keychain; never print the token. Paste this command into the generated prompt so the night session has it even without project memory.

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

# Overnight prompt for PROJECT VERSION (paste into a fresh Fable session)

FIXED. Build VERSION overnight while I sleep. I am AFK the whole night and wake around WAKE_TIME TIMEZONE. Do not ask me anything: decide alone, log each decision, and keep working until the usage ceiling or the deadline below. Stopping early wastes the budget I set aside for this.

FIXED. You are the orchestrator. Your own context is expensive (you run on Fable, which has its own limit), so stay lean: plan, delegate, check results, keep the tracker. Let subagents do the reading of large files, the coding, the tests and the screenshots.

## Setup, first 15 minutes

FIXED, with names filled:

1. Run `date`. Use real clock times for every stamp all night; never guess the time.
2. Arm the resume job before anything else: a CronCreate job every 30 minutes with the text "continue the VERSION overnight plan: run date, check usage, read TRACKER/status.md, resume dead agents, pick up the next open item". A rate-limited session never wakes by itself; this job is what wakes it.
3. Check usage with the command below (paste the usage command). Write the numbers and reset times into `status.md`.
4. **Preflight, while I am still at the computer.** Run one harmless command of every kind the night needs, so every approval prompt appears now (FILL from the project: the keychain read in the usage command, one test file per test suite, any server or program the run starts, one screenshot, a `git commit --dry-run`, a `git push --dry-run` if the run pushes, the deploy tool in a harmless form, one Agent call per model, a CronCreate). Then tell me in one short list which ones asked for approval. End that message with this banner in a code block, and in the same turn run `say "Preflight done. You can go now."` and `osascript -e 'display notification "You can go now" with title "PROJECT VERSION" sound name "Glass"'`:

```
 __   __ ___  _   _     ____    _    _   _     ____   ___      _   _  ___ __        __
 \ \ / // _ \| | | |   / ___|  / \  | \ | |   / ___| / _ \    | \ | |/ _ \\ \      / /
  \ V /| | | | | | |  | |     / _ \ |  \| |  | |  _ | | | |   |  \| | | | |\ \ /\ / /
   | | | |_| | |_| |  | |___ / ___ \| |\  |  | |_| || |_| |   | |\  | |_| | \ V  V /
   |_|  \___/ \___/    \____/_/   \_\_| \_|   \____| \___/    |_| \_|\___/   \_/\_/
```

   If something still needs me, show no banner: say what I must do, first and in bold. Do not wait for my answer after the banner; go on.
5. Read `CLAUDE.md`, the README, the memory notes about earlier overnight runs, and in full FEEDBACK_FILES. Say which lines count as requests and name any screenshots to look at.
6. Branch: `git checkout -b BRANCH` from BASE_BRANCH (or stay on the open branch for a pass). Commit nothing on any branch.
7. Tracker at TRACKER with `spec.md`, `status.md`, `decided-alone.md`, `issues/NN-slug.md`. Every request becomes an issue before code is written. Name the screenshot folder, test file prefix and tool prefix for this version.

## Usage limits: this decides when you stop

FILL the snapshot line: at TIME the numbers were weekly N% (resets WHEN), Fable-only N%, 5-hour N% (resets WHEN). Re-read them yourself.

FIXED, with the numbers computed from the budget ceiling (N) in SKILL.md. When Fable use is low, add a line that the Fable-only figure has its own lower ceiling and the same thresholds are applied to it, each 10 points lower.

- **Weekly ceiling.** Use as much of the budget up to N% as you can on real work. Hitting the ceiling mid-task leaves broken code and no summary, which is the worst outcome of the night.
  - At **N minus 6** on either the weekly or the Fable-only figure: stop starting new features. Finish what is running.
  - At **N minus 4**: start the wrap-up: the final Fable review, its fixes, the full test suite, the summary.
  - At **N**: all writing is done. Only the cron cleanup remains. Never go past N plus 1 (at most 99%).
  - Check usage every hour and before every wave. Estimate a wave's cost from the previous one and do not start a wave that would cross the next threshold.
- **5-hour windows.** Work straight through them. When the 5-hour figure passes 85%, add a one-shot CronCreate for its reset time plus 3 minutes. If a 429 error appears, read the reset time from its text and add a one-shot CronCreate for that time plus 2 minutes. After any reset: run `date`, check usage, resume every dead subagent with SendMessage (they keep their transcript), then run the test suite before anything else, since agents may have died mid-edit.
- **Weekly reset during the night.** FILL only if the weekly count resets before WAKE_TIME: the wrap-up must be done before the reset if the ceiling came first; after the reset, continue until WAKE_TIME spending at most 8% of the new week, then refresh the summary.

## Subagents: which model for what

FIXED. Pass `model` on every Agent call. Each parallel agent owns disjoint files and re-reads a file before editing it.

| Model | Use it for |
|---|---|
| **Fable** | FILL from the Fable use level in SKILL.md, naming tonight's actual hard bugs and features. Always end the cell with: the Fable-only limit is shared with the orchestrator. |
| **Opus** | Most feature work and ordinary bug fixes; anything touching core state, rendering or event handling. The read-only review after each wave. |
| **Sonnet** | Mechanical work with a clear spec: wording, tooltips, layout tweaks with a screenshot to match, screenshot runs, docs, tests for a finished module. Tell Sonnet agents to finish in one go and not end their turn while a background command runs. |

## Rules that are not negotiable

FILL from the project's `CLAUDE.md`, memory and last night's prompt: data safety, test-suite discipline, screenshot rules, house style, writing rules (no em-dashes, no YAML frontmatter, no angle brackets in markdown).

## Wave 1: bugs and faults I reported

FILL.

## Wave 2: changes I asked for

FILL.

## Wave 3: items I left blank, decide for me

FILL: list the blank items; tell the session to try each with real data, fix what is broken or clumsy, and not redesign what works. Choices left unticked without a note stay as they are and go into the morning questions.

## Wave 4 and onward: work to delight me in the morning

FIXED, with project-fitting examples: when waves 1 to 3 are done, reviewed and green, and budget allows, keep going and choose the work. Directions: measured speed gains (before and after numbers), fewer clicks and full keyboard use in common flows, robustness on odd inputs, polish that shows in daily use, something small the notes hint at. Each gets an issue, a `decided-alone.md` line, tests and a screenshot. Prefer finished and solid over large and half-done. Name the areas that wait for owner decisions and are off-limits.

## Wrap-up: at N minus 4 usage, or by WRAP_TIME (30 minutes before WAKE_TIME), whichever comes first

FIXED:

1. **Final Fable review** (one Fable subagent) of the whole diff from BASE_BRANCH to the working tree: bugs, regressions, dead code, wording. Fix what it finds with Opus agents, or Fable for anything hard.
2. Full test suite, alone, green. Put the count in the summary.
3. `TRACKER/morning-summary-YYYY-MM-DD.md` (wake date): what shipped with screenshots, what was decided alone, what was not done and why, measurements, usage at start and end, "Questions for you", a suggested commit split with messages in the project's commit style and no trailers.
4. Update the README and user guide.
5. A fresh tryout file for the owner in the shape of the previous one, holding only what needs their opinion; move answered notes into the record file.
6. Save a short memory note about the night: what was built, what went wrong, lessons.
7. Delete every cron job created tonight. Then stop.
