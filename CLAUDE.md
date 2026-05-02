# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-file Claude Code skill called `handoff`. It generates a copy-paste-ready markdown snippet for transferring session context to a fresh Claude Code session. Flat layout — `SKILL.md` lives at the repo root, not under `skills/`.

## Layout

- `SKILL.md` — the skill (frontmatter + workflow).
- `README.md` — user-facing docs.
- `LICENSE` — MIT.

## Iterating on the skill

Edit `SKILL.md`, then trigger the skill in any Claude Code session by saying `/handoff`, `/handoff full`, or `/handoff quick`. If installed via symlink there's no reload step. Iterate against real conversations rather than synthetic test prompts — the skill is meant to compress session context, so it can only really be evaluated on a session that has context worth compressing.

## Design constraints — preserve these

Deliberate choices — don't undo on autopilot.

- **No disk writes.** Handoff lives in Claude's reply inside a 4-backtick fence. The snippet model exists so the user can paste into a different repo, machine, or web Claude — on-disk handoffs don't help there.
- **No resume-side mechanism.** No `/handoff:resume`, no auto-detection. The snippet is self-describing markdown — the user pastes it and tells the next Claude to continue.
- **Args, not subcommands.** `/handoff full` and `/handoff quick`, not `/handoff:full` etc. Single SKILL.md, default `full`.
- **Opt-in inclusion.** Every section earns its place by giving the next session something they can't trivially derive from git/code. Default omit. Failed Approaches when there were any — no "None — happy path" padding. Self-review pass before fencing keeps Claude out of template-fill mode.
- **Both modes lead with a one-line disclaimer.** Quick: "trimmed for brevity, may miss context." Full: "verify anything load-bearing before acting." Keep them short — without the framing, the next session reads the snippet as spec and stops thinking critically.
- **English content.** Snippet template, README, and SKILL.md instructions are English. Trigger phrases are English (the user uses the word "handoff" verbatim regardless of conversational language).
