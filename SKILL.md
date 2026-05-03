---
name: handoff
description: Copy-paste-ready handoff snippet for transferring context to a fresh Claude Code session. `/handoff` for full (default), `/handoff quick` for ~10 lines. Delivered in a 4-backtick markdown block; nothing written to disk.
---

# Handoff

Produce a snippet the user can paste into a fresh Claude Code session — a different repo, machine, or web Claude. Lives in your reply only.

`full` (default): runs as long as the content actually needs. `quick`: ~10 lines.

## Workflow

Judgment exercise, not template fill. The failure mode is a polished snippet that mostly restates what the next session could read off `git status`. Default for every section is *omit*.

### 1. Check git (optional for trivial quicks)

Run in parallel, tolerate failure silently:

- `git status --short`
- `git branch --show-current`

Use the output to sanity-check your claims, not as snippet content.

### 2. Reconstruct, then keep only what earns its place

Mentally reconstruct: the underlying goal, what was tried and why it failed, what's actually true vs. just discussed, the obvious next move, anything that would silently bite the next session.

For each candidate piece of content, ask:

> *Will the next session waste time, repeat a mistake, or miss the goal without this?*

If no — omit. A section earns inclusion only when it gives the next session something they can't trivially derive from git, the codebase, or a quick re-read.

**Usually earns it:** Goal, Next, Failed Approaches (when there were any), non-trivial code shapes, non-default config, key decisions whose rationale isn't obvious from the code.

**Usually doesn't:** Files-to-know tables that restate `git status`. Standard env vars the README covers. Numbered resume instructions when one sentence does. "None — happy path" padding under empty headers.

### 3. Self-review

- `full`: cut anything the next session could do without.
- `quick`: missing load-bearing context? Add it. Padding inside the budget? Cut it.

## Reply format

Three parts, in order:

1. One line above: `Snip ready — copy the block below into your new session:`
2. A code block fenced with **four** backticks, language tag `markdown`, containing the snippet. (Four-tick outer fence so 3-tick blocks inside — code, JSON, shell — render correctly when pasted.)
3. One line below: `Anything to adjust before you copy?`

## Format: full

**Required:** title `# Handoff: [task]`, disclaimer blockquote.
**Almost always:** Goal, Next.
**Opt-in (judgment call):** Failed Approaches, Key Decisions, Code Context, Current State, Warnings, Setup. Section names and granularity are your call — let content drive structure. Use `##` headers for 3+ chunks; inline `**Label:**` for shorter material.

**Disclaimer text** (verbatim):

> Handoff written from session memory. Verify anything load-bearing before acting.

**Example** — illustrative, not a template. Most candidate sections were considered and dropped.

````markdown
# Handoff: Idempotent webhook handler

> Handoff written from session memory. Verify anything load-bearing before acting.

**Branch**: feat/webhook-idempotency  
**Status**: Blocked on dedupe strategy

## Goal

Make `/webhooks/upstream` idempotent so the provider's automatic retries don't trigger duplicate downstream side effects.

## Next

Pick a dedupe strategy: a `processed_events` table with a unique constraint on the upstream event ID, vs. Redis `SETNX` with a 24h TTL. Wire it in ahead of the existing dispatch.

## Failed Approaches

- Unique constraint on `orders.upstream_id` blocked legitimate updates because some events don't map 1:1 to orders.
- Postgres advisory lock keyed on event ID worked, but held connections during downstream sends and starved the pool under load.

## Warnings

- `WEBHOOK_SECRET` rotates weekly. Local `.env` is last week's; tests use a fixture. Prod-shape signatures fail locally.
````

## Format: quick

~10 lines. Same opt-in rule.

**Required:** title, disclaimer blockquote.
**Almost always:** **Goal**, **Done**, **Next** as inline labels.
**Opt-in:** **Watch out**, **Failed approaches** — only when there's something real.

**Disclaimer text** (verbatim):

> Quick handoff — trimmed for brevity, may miss context.

**Example:**

````markdown
# Handoff: Flaky upload test on macOS only

> Quick handoff — trimmed for brevity, may miss context.

**Goal**: Find why `tests/upload.test.ts` passes on Linux CI but fails ~30% of local macOS runs.
**Done**: Reproduced locally; narrowed to multipart parsing, not the upload itself.
**Next**: Re-run with `DEBUG=busboy:*` and check whether boundary header parsing differs across macOS and Linux Node 20 builds.
**Watch out**: Test uses real disk I/O (no temp-fs mock) — failures may be filesystem-timing, not code.
````
