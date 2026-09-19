# away-from-keyboard

Two skills for [Claude Code](https://claude.com/claude-code) that write the instructions for a fresh session to work alone for hours while you sleep or are away.

You run one of them in your current session. It writes one prompt file into your project and gives you a single line to paste into a new session. The new session then works without asking you anything. It hands the work to helper agents, keeps an eye on your usage limits, starts itself again after a limit resets, and leaves a summary for when you come back.

> **Claude Code** is Anthropic's coding assistant. A **skill** is a set of instructions you can hand it on demand, by typing a `/` command. A **session** is one conversation with Claude Code. **Helper agents** (Claude Code calls them subagents) are extra copies of Claude that one session starts to do parts of the work in parallel.

## The skills

| Command | What it does |
|---|---|
| `/mock-work-all-night` | Reads your feedback notes on the project, sorts every request into bugs, changes and open questions, and writes an overnight prompt that works through them in order, then keeps improving the project until your budget ceiling or your wake time. |
| `/mock-prompt-new-session-to-work-on-the-plan` | For a plan you have already written together with Claude, in a tracker folder. It writes a prompt that hands that plan to a fresh session, and records how much usage and time the plan cost. |

Use the first when you have a list of notes and want the night used fully. Use the second when the plan is already written and should simply be carried out. Neither skill does the work itself. Each only writes the prompt.

Both skills start only when you type the command. Claude Code never starts them on its own.

## What you need

- **[Claude Code](https://claude.com/claude-code)** on a paid Claude plan. The prompts are built around the plan's usage limits: a weekly limit, a 5-hour limit, and a separate weekly limit for the Fable model.
- **Access to Fable**, Anthropic's most capable Claude model. The long-running session runs on it, and hands the hardest parts to it.
- **A Mac.** To check your usage, the prompt includes a short command that reads your Claude Code login from the macOS keychain (the Mac's password store). It needs `curl` and `python3`, which come with macOS. On other systems, the rest of the prompt still works, but the session cannot read its usage.
- **A project in a git repository.** The session creates a branch and works on it.

Two warnings before you start:

- **The overnight prompt spends your budget on purpose.** By default it keeps working until 98% of your weekly usage is gone. You can set a lower ceiling (see [Using it](#using-it)).
- **The usage command uses an address that Anthropic does not document** (`api.anthropic.com/api/oauth/usage`). It may stop working without notice. The command never prints your login token.

## Installing

Options 1 to 3 are for Claude Code. Option 4 is for other agents such as Codex or Gemini CLI. Pick one.

### Option 1 — ask Claude Code to do it

This is the easiest way, and you do not need a terminal. Open Claude Code and paste this:

```
Please install the Claude Code skills from https://github.com/panuakdet-gmail/mock-away-from-keyboard.

1. Download the repository, with its full history, to a temporary folder.
2. For each folder inside its skills/ folder, look for a folder with the same name in ~/.claude/skills/.
   - If there is none, copy the folder there.
   - If there is one, and every file in it matches a version that was once published in this repository, replace it with the new version.
   - If there is one, and it matches no published version, I have edited it. First move my copy to ~/.claude/skill-backups/<folder name>-<today's date>/. Then install the new version, show me what I had changed, and offer to merge my changes into it.
3. Delete the temporary download and tell me what you installed or updated.
```

Claude Code asks your permission before it changes anything. When it has finished, restart Claude Code.

**To update later**, paste the same prompt again. If you changed a skill yourself, your version is saved first, and Claude Code offers to merge your changes into the new one.

### Option 2 — install as a plugin

Choose this if you want Claude Code to handle updates for you. Inside Claude Code, run:

```
/plugin marketplace add panuakdet-gmail/mock-away-from-keyboard
/plugin install away-from-keyboard@away-from-keyboard-skills
```

To update later, run `/plugin marketplace update away-from-keyboard-skills`. Do not also use Option 1, otherwise every skill is installed twice.

### Option 3 — copy the folders yourself

In a terminal:

```bash
git clone https://github.com/panuakdet-gmail/mock-away-from-keyboard.git
mkdir -p ~/.claude/skills
cp -R mock-away-from-keyboard/skills/* ~/.claude/skills/
```

Then restart Claude Code. To update, delete the downloaded `mock-away-from-keyboard` folder and run the same commands again. This replaces the installed folders, so any changes you made to them are lost.

### Option 4 — ask another agent to do it

Open Codex, Antigravity, Gemini CLI, or whatever agent you use, and paste this:

```
Please install the skills from https://github.com/panuakdet-gmail/mock-away-from-keyboard for this agent.

1. Download the repository, with its full history, to a temporary folder.
2. Each folder inside its skills/ folder is one skill. Install each folder whole, including subfolders such as assets/, because the instructions use those files.
3. The skills were written for Claude Code, so adapt only the packaging: register each one the way this agent handles reusable skills or commands, so I can run it by name. Do not change the instructions inside each SKILL.md.
4. Before you install each skill, check whether I already have it.
   - If I do not, install it.
   - If I do, and its files match a version that was once published in this repository, replace it with the new version. Ignore packaging changes that an agent made when it installed the skill.
   - If I do, and it matches no published version, I have edited it. First copy my version to a backup folder outside the place this agent loads skills from, and tell me where it is. Then install the new version, show me what I had changed, and offer to merge my changes into it.
5. Delete the temporary download. Then tell me what you installed or updated, and how to run each skill here.
```

**To update later**, paste the same prompt again. The prompts these skills write rely on Claude Code features, such as scheduled wake-ups and helper agents on a chosen model, so they may not run fully in another agent.

## Using it

### Overnight, from your notes

Run it in your project, in the session where you have been working. Everything after the command is optional, in plain words and in any order:

```
/mock-work-all-night
/mock-work-all-night notes/feedback-v4.md, wake at 7:30
/mock-work-all-night budget=85 fable=low
```

- **`budget=N`** (50 to 99, default 98) is the highest weekly usage the night may reach. New features stop 6 points below it, and the wrap-up starts 4 points below it.
- **`fable=low`, `fable=normal` or `fable=high`** (default `normal`) sets how much of the work goes to Fable helper agents. `low` also gives Fable its own ceiling, 10 points below the budget.

It finds your feedback notes, the version name and the wake time from the project and from Claude's memory when you do not give them. It asks you only if it cannot find any feedback at all.

### A plan you already wrote

Run it at the end of the session in which you and Claude wrote the plan:

```
/mock-prompt-new-session-to-work-on-the-plan
/mock-prompt-new-session-to-work-on-the-plan -tracker/export-rework, done by Monday 9:00
/mock-prompt-new-session-to-work-on-the-plan budget=90 fable=high
```

It expects the plan in a tracker folder, `-tracker/NAME/`, holding a `spec.md` (the problems and your decisions), a `brief.md` (the facts the work needs, such as paths and measurements) and an `issues/` folder with one file per piece of work. If something is missing, it fixes the tracker before it writes the prompt.

- **`budget=N`** defaults to 95. It defaults to 100 when your weekly usage resets before the work is expected to end, because unused budget is lost at the reset. The session still spends only what the plan needs.
- **`fable=low`, `fable=medium` or `fable=high`** (default `medium`). Note that this skill says `medium` where the overnight skill says `normal`.

### Starting the run

Both skills end by showing you one line in a box, like this:

```
Read -docs/prompts/2026-09-14-v4-overnight-prompt.md in full and carry it out tonight. I'm asleep until morning: don't ask me anything, decide alone, and start with the setup steps.
```

Open a new Claude Code session on Fable, paste only that line, and leave. Do not paste the contents of the prompt file.

## What it produces

In your project:

```
-docs/prompts/
└── 2026-09-14-v4-overnight-prompt.md    the prompt file the new session reads
```

The new session then keeps its own records while it works:

```
-tracker/NAME/          (the overnight skill picks the folder; the plan skill uses your existing one)
├── spec.md             the problems and your decisions
├── status.md           progress of every piece of work, and a usage log added to about every hour
├── decided-alone.md    every choice the session made without you, with the reason
├── issues/             one file per request or piece of work
└── summary file        what shipped, what was not done and why, questions for you,
                        usage and time spent, and a suggested way to split the commits
```

The session never commits. You review the work in the morning and commit it yourself.

## Credits

Written from scratch by its author. It bundles no third-party code or text, and implements no named method that belongs to someone else.

## Licence

MIT — see [LICENSE](LICENSE). It covers the text in this repository. It does not cover the prompts the skills write for your projects; those are yours.
