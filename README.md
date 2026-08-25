# manual-qa-loop

> A portable [Agent Skill](https://agentskills.io) that turns "test the app thoroughly" into a repeatable process: explore it like an expert QA tester, fix any real bugs with a regression test, write a dated report, and keep running rounds until three in a row find nothing. Works in Claude Code, Cursor, Codex CLI, and any other Agent-Skills-compatible tool.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Part of the [Agent Skills catalog](https://github.com/code-with-rashid/agent-skills) — browse all skills and their install steps for every tool.

## The problem

"Test this thoroughly" from an AI agent usually means one pass: click around a bit, maybe file a couple of issues, declare it done. There's no explicit stop condition, no persistence across rounds, and no forcing function to keep testing areas that weren't already covered.

## What this does

- Runs manual, exploratory end-to-end QA against a *running* app — not a static code read.
- Explores breadth-first across rounds: golden paths, edge cases, cross-feature consistency, gating/permission enforcement, validation UI, accessibility basics.
- Root-causes and fixes real bugs it finds, each with a regression test that fails before the fix and passes after.
- Writes a dated report every round (`qa/manual/round-<N>-report.md`) so progress and prior findings persist across sessions.
- Has an explicit, objective stop condition — the default is 3 consecutive clean rounds — plus a hard cap so it never silently loops forever.

This is a different technique from automated/instrumented hardening (coverage, mutation testing, fuzzing) — see [adversarial-qa](https://github.com/code-with-rashid/claude-adversarial-qa-skill) for that. This skill simulates a human QA tester driving the actual UI; that one drives code-level test instrumentation. They're complementary, not overlapping.

## Install

### Recommended: as a plugin

Inside a Claude Code session, in any project:

```
/plugin marketplace add code-with-rashid/manual-qa-loop
/plugin install manual-qa-loop@manual-qa-loop
```

Then run `/reload-plugins` (or start a fresh `claude` session) — a plugin installed
mid-session doesn't retroactively load into that session's context, so the skill
won't be visible yet until you do. Confirm with `/plugin list`, or `claude plugin list`
from a regular terminal.

By default this installs at **user scope** (available in every project). To scope it
to just the current project instead, run the equivalent from a terminal:

```bash
claude plugin marketplace add code-with-rashid/manual-qa-loop --scope project
claude plugin install manual-qa-loop@manual-qa-loop --scope project
```

To remove it later: `/plugin uninstall manual-qa-loop@manual-qa-loop`.

<details>
<summary>Alternative: manual copy (no plugin system)</summary>

This is a folder with a `SKILL.md` — no build step, no dependencies. **Clone this
repo first**, then copy `skills/manual-qa-loop/` into a `.claude/skills/` directory.

#### macOS / Linux (bash/zsh)

```bash
git clone https://github.com/code-with-rashid/manual-qa-loop.git ~/manual-qa-loop

# per-project (recommended)
mkdir -p /path/to/your-repo/.claude/skills
cp -r ~/manual-qa-loop/skills/manual-qa-loop /path/to/your-repo/.claude/skills/

# per-user (available in every project, instead of per-project)
mkdir -p ~/.claude/skills
cp -r ~/manual-qa-loop/skills/manual-qa-loop ~/.claude/skills/
```

#### Windows (PowerShell)

```powershell
git clone https://github.com/code-with-rashid/manual-qa-loop.git $HOME\manual-qa-loop

# per-project
New-Item -ItemType Directory -Force -Path "C:\path\to\your-repo\.claude\skills" | Out-Null
Copy-Item -Recurse -Force "$HOME\manual-qa-loop\skills\manual-qa-loop" "C:\path\to\your-repo\.claude\skills\"

# per-user
New-Item -ItemType Directory -Force -Path "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse -Force "$HOME\manual-qa-loop\skills\manual-qa-loop" "$HOME\.claude\skills\"
```

</details>

### Other tools (Cursor, Codex CLI, ...)

This skill is a standard `SKILL.md` package with no Claude-Code-only dependencies, so
it installs the same way any Agent Skill does. See the
[catalog's install guide](https://github.com/code-with-rashid/agent-skills#install)
for exact steps in Cursor, Codex CLI, and other tools.

## Use

Open your agent in the target repo with the app running, and either ask for it in
plain language ("test the app thoroughly", "find and fix bugs end to end", "run a QA
loop") or invoke it directly:

```
/manual-qa-loop
```

It figures out how to run the app, how to verify a fix (lint/test/build), and where
to write reports — then loops rounds of explore → fix → verify → report until it hits
the stop condition (default: 3 consecutive clean rounds).

## License

MIT
