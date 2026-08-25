---
name: manual-qa-loop
description: Run iterative end-to-end manual QA on a running app - explore it like an expert tester, write a dated report each round, fix any real bugs found (with regression tests), and keep repeating rounds until two in a row find zero issues. Use when the user asks to "test the app thoroughly", "find and fix bugs end to end", "run a QA cycle/loop", or similar open-ended quality passes over an app.
version: 1.0.0
license: MIT
---

# Manual QA loop

A repeatable, framework-agnostic process for hardening an app through successive
rounds of manual end-to-end testing. Each round: explore, find, fix, verify,
report. The loop only ends when two consecutive rounds find nothing.

This skill runs as your normal foreground work (not a delegated subagent) -
it needs the browser/app open, needs to edit real source files, and needs to
carry findings from one round into the next.

## 0. Setup (once, before round 1)

Figure out, don't assume:

- **How to run the app.** Check `.claude/launch.json` for an existing dev-server
  config; otherwise look for the obvious entry point (`package.json` scripts,
  `pubspec.yaml`, `Cargo.toml`, a `README`/`CLAUDE.md` "getting started"
  section). If your tool has a built-in browser/preview capability (e.g.
  Claude Code's Browser pane `preview_start`/`preview_start {name}`), use that
  rather than a raw background shell server, so you get a real browser you can
  drive. Otherwise, start the dev server yourself and use whatever
  browser-automation or preview capability your tool provides, or ask the user
  to open it and confirm the URL.
- **How to verify a change.** Find this project's own lint/typecheck, test,
  and build commands (e.g. `flutter analyze` / `flutter test`, `npm run
  lint` / `npm test`, `cargo check` / `cargo test`, etc.) by reading its
  config files - don't hardcode one stack's commands.
- **Where reports go.** Default to `qa/manual/round-<N>-report.md` under the
  project root. If that directory already has `round-*-report.md` files from
  a prior invocation of this loop, treat this as a continuation: read the
  most recent 1-2 reports to see what's already been covered and what
  patterns were already fixed, and start numbering at the next integer.
- **Stop threshold.** Default: stop after **3 consecutive rounds** with zero
  new findings. If the user's request specifies a different number, use that
  instead.
- **Hard cap.** Regardless of the stop threshold, never silently run past
  ~25 rounds. If you hit the cap without satisfying the stop condition, stop,
  report status honestly, and ask how to proceed - don't keep looping forever
  on a moving target.

## 1. Each round

Repeat this whole section per round, back to back, without waiting for the
user's go-ahead between rounds - the point of the loop is to keep running
until the stop condition is met. Only pause for the user if you hit a real
blocker (can't start the app, ambiguous scope, need a decision only they can
make).

### a. Explore like an expert QA tester

Pick an area of the app you (or a prior round, per its report) haven't
exercised yet, and prefer breadth across rounds over re-testing the same
screen repeatedly. For whatever you touch, actually exercise it - don't just
read the code and assume it works:

- Golden path first, then edges: empty states, boundary values, invalid
  input, rapid repeated actions, browser/app back-and-forth navigation.
- Cross-feature consistency: does this screen agree with what other screens
  show for the same underlying data? Look for state that two places compute
  differently (a classic source of real bugs - see the methodology notes
  below).
- Anything gated (paywalls, permissions, feature flags, tiers): confirm the
  gate is actually enforced everywhere it should be, not just in the one
  obvious entry point.
- Validation and error UI: does an error message ever get left on screen
  after the underlying problem is already fixed?
- Accessibility basics: do interactive controls have accessible
  names/tooltips, not just an icon?

When something looks wrong, don't stop at the symptom - read the relevant
source to find the actual root cause before deciding it's a real bug (see
methodology notes: a lot of apparent bugs during live UI testing are actually
artifacts of the testing method, not the app).

### b. Fix what's real

For each confirmed bug:

1. Root-cause it via source reading, not guessing.
2. Apply the smallest fix that addresses the actual cause - no unrelated
   refactoring or scope creep.
3. Add a regression test that locks in the exact scenario you reproduced.
4. Run the project's own verification commands (lint/typecheck, full test
   suite, build) and confirm they're clean.
5. Re-verify live: reproduce the original symptom against a fresh build/reload,
   confirm it's gone.

### c. Write the round's report

Always write a report for the round, whether or not you found anything -
`qa/manual/round-<N>-report.md`. Keep a consistent shape so future rounds (and
the user) can skim it:

```markdown
# Manual E2E QA - Round <N> (<date>)

<1-3 sentences: what this round covered and why (e.g. "areas not yet exercised in prior rounds", or the specific feature that prompted this pass).>

## Findings

### <N>. <short title> - FIXED

- **Where:** <file(s), linked>
- **Symptom:** <what you actually observed, reproduced live - not theoretical>
- **Root cause:** <the real mechanism, from reading the source>
- **Fix:** <what changed and why that's the minimal correct fix>
- **Verification:** <lint/test/build results, live re-verification>
- **Regression test:** <what was added, where>

(omit this section entirely if nothing was found this round)

## Investigated, no bug found

<Anything you dug into that turned out fine - worth recording so future
rounds don't waste time re-checking it, and so a confusing false lead you
chased down is documented rather than silently discarded.>

## Verification

<Commands run and their results this round; note "no code changes" if none.>

## Verdict

<One or two sentences: how many real bugs this round, and - once you reach
the stop condition - an explicit statement that this is the Nth consecutive
clean round satisfying it.>
```

### d. Manage context between rounds

Each round should be self-contained: don't re-read the whole codebase or
replay prior rounds' exploration from memory - use each round's own report
file as the record instead of keeping it all live in context. If a session
tool for marking a chapter boundary is available, use it at the end of each
round to keep the transcript navigable.

Interactive context-compaction commands (e.g. Claude Code's `/compact`) are
typically client-side and only the user can run them - don't claim to run
one yourself. If you're in an interactive session and context is visibly
getting large, you can mention that the user may want to trigger their
tool's context-management feature before the next round; otherwise just keep
going. Many tools, including Claude Code, apply some form of automatic
compaction between turns that's designed to preserve continuity across
exactly this kind of multi-round loop - rely on that rather than manual
context management where your tool supports it.

## 2. Stopping

After each round's report is written, check the stop condition: N
consecutive rounds (default 3) with an empty "Findings" section. As soon as
it's met, stop looping and give the user a concise wrap-up: total rounds run,
every real bug found and fixed (one line each, file-linked), and confirmation
that the final N rounds were clean. Don't commit or push anything - that's a
separate, explicit ask per this project's own conventions.

## Methodology notes (apply throughout)

- **A live UI "bug" is sometimes the tester's own doing.** Repeated
  scroll/drag/tap actions at a fixed nominal coordinate can land on different
  actual widgets once a scrollable container's offset has shifted underneath
  them - producing what looks like unstable app state but is really stray
  input. Before concluding something is broken, re-verify with a single clean
  sequence: fresh navigation -> one deliberate action -> one screenshot, no
  extra interactions in between.
- **Screenshots can lag one beat behind a navigation.** If a screenshot right
  after a navigating click looks stale, wait 1-2s and re-screenshot before
  concluding the app didn't update.
- **Prefer the pattern already in the codebase over inventing a new one.**
  When a bug is "this component doesn't follow the consistency rule every
  sibling component already follows," fix it by applying that same existing
  pattern rather than designing a new mechanism.
- **A missing regression test means the bug will come back.** Every fix in
  this loop should be accompanied by a test that would fail on the
  pre-fix code and passes after.
