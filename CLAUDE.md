# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-file Claude Code skill called `handoff`. It generates a copy-paste-ready markdown snippet for transferring session context to a fresh Claude Code session. Flat layout — `SKILL.md` lives at the repo root, not under `skills/`.

The repo is also symlinked into the user's local skills directory:

```
/Users/sebm1/.claude/skills/handoff -> /Users/sebm1/handoff
```

Edits to `SKILL.md` here are immediately live for any Claude Code session — no install/deploy step.

## Layout

- `SKILL.md` — the skill itself (frontmatter + workflow). Loaded into Claude Code when the skill triggers.
- `README.md` — user-facing docs for someone considering installing this from GitHub.
- `LICENSE` — MIT.

That's the whole repo. There are no scripts, no build, no tests, no `evals/` directory yet.

## Iterating on the skill

Edit `SKILL.md`, then trigger the skill in any Claude Code session by saying `/handoff`, `/handoff full`, or `/handoff quick`. The symlink means there's no reload step. Iterate against real conversations rather than synthetic test prompts — the skill is meant to compress session context, so it can only really be evaluated on a session that has context worth compressing.

If you add formal evals later, follow the skill-creator pattern: `evals/evals.json` with realistic prompts, results into a sibling `handoff-workspace/iteration-N/` directory.

## Design constraints — preserve these

These were deliberate choices made during the initial design grilling. Don't undo them on autopilot.

- **No disk writes.** The handoff lives entirely in Claude's reply, inside a 4-backtick markdown fence. Adding a `HANDOFF.md` writer or `pbcopy` step defeats the whole point — the snippet model exists so the user can paste into a different repo, machine, or web Claude where on-disk handoffs don't help.
- **No resume-side mechanism.** No `/handoff:resume` command, no auto-detection of pasted snippets. The snippet is self-describing markdown; the user pastes it and tells the next Claude to continue. Nothing more.
- **Args, not subcommands.** `/handoff full` and `/handoff quick`, not `/handoff:create` and `/handoff:quick`. Single SKILL.md, parsed args, default to `full`.
- **Failed Approaches always appears in `full`.** Even when nothing failed (`None — happy path so far.`). The explicit "None" tells the next session you considered it. Other empty sections are dropped.
- **English content.** Snippet template, README, and SKILL.md instructions are English. Trigger phrases are English (the user uses the word "handoff" verbatim regardless of conversational language).

## Reference

Snippet structure and "failed approaches are mandatory" principle inspired by [willseltzer/claude-handoff](https://github.com/willseltzer/claude-handoff). The key divergence is the no-disk-write model — that repo writes a `HANDOFF.md` file; this skill returns a copy-paste block instead.
