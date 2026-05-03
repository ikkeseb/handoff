---
name: handoff
description: Generate a copy-paste-ready handoff snippet so a fresh Claude Code session can pick up where this one left off. Trigger on "/handoff", "/handoff full", "/handoff quick", "create handoff", or any user message containing "handoff". Also offer (one-line, end of reply) when the user signals a session transfer or context pressure — phrases like "starting a new session", "context is full", "fresh context", "switching to a fresh agent". Writes nothing to disk; the snippet lives inside a 4-backtick markdown block in the reply. Args — `quick` for a tight ~10-line briefing, `full` (default) runs as long as content actually needs. Sections are opt-in via judgment, not template-fill.
---

# Handoff

Generate a copy-paste-ready snippet that lets a fresh Claude Code session pick up where this one left off — without writing any files. The snippet lives in your reply (a 4-backtick markdown block) and nowhere else, so the user can paste it into a different repo, machine, or web Claude.

## When to invoke

**Direct triggers:** `/handoff`, `/handoff full`, `/handoff quick`, "create handoff", "lag handoff", or any user message containing "handoff". Default to `full`; switch to `quick` only if the invocation contains "quick".

**Pause signals:** if the user mentions a session transfer or context pressure ("starting a new session", "context is full", "fresh context"), offer once at the end of your normal reply — "Want a handoff snip before you start fresh?" — and drop it if ignored. Don't auto-generate.

**Don't trigger on:** vague pause cues like "I need a break" (could mean anything) or simple milestone completions. A handoff is for session transfer, not checkpointing.

Both modes share the opt-in inclusion rule below. The only difference is budget: `quick` aims for a tight ~10-line briefing; `full` runs as long as the relevant content actually needs.

## Workflow

This is a judgment exercise, not a template fill. Spend tokens *before* the snippet — deliberating about what this specific session actually needs — rather than after, on padding. The most common failure mode is producing a polished-looking snippet that mostly restates things the next session could read off `git status` in three seconds. Resist that.

### 1. Gather signals

Run these in parallel as a baseline, tolerating failure silently. Outside a git repo, skip without comment.

- `git status --short` — anchors which files were actually edited vs. just discussed.
- `git branch --show-current` — populates the Branch line if you include one.

Add `git log --oneline -5` only if the snippet will reference recent commits. Add `git diff --stat` only if you specifically need a sense of change scope; usually `status --short` covers it.

The point is to anchor your claims in ground truth instead of memory. Models often misremember which files were actually edited vs. just discussed; git output corrects that. Don't pad the snippet with these outputs verbatim — the next session can run the commands themselves. Use them to sanity-check what *you* are about to claim.

### 2. Reconstruct, then decide what earns its place

Before reaching for any section header, mentally reconstruct where this session got to:

- **What is the user really trying to accomplish?** Not the immediate ask — the underlying goal.
- **What did we try, in order, and what didn't work, and why?** The "why" is usually the load-bearing part.
- **What is actually true vs. just discussed?** Git output is the tiebreaker.
- **What is the obvious next move, and why is it obvious?** If you can't say why, the next session won't know either.
- **What would silently bite the next session?** Non-default config, half-applied migrations, an intentional-looking-wrong flag, a skipped test.

That reconstruction is the source material. For each candidate piece of content, ask:

> *Will the next session waste time, repeat a mistake, or miss the goal without this?*

If no — omit. Default for every section is "leave out." A section earns inclusion only when it gives the next session something they can't trivially derive from git, the codebase, or a quick re-read.

**Usually earns its place:** Goal and Next (almost always). Failed approaches when there were any — "Tried X, failed because Y, switched to Z" saves hours; don't pad with "None — happy path", absence is itself the signal. Code context when there are signatures or shapes that aren't easy to re-derive. Key decisions when the rationale isn't obvious from the code. Warnings/setup/current state when there's something real and non-default to flag.

**Usually doesn't:** "Files to Know" tables that restate `git status --short`. "Working / Broken" subsections that paraphrase the code. Numbered "Resume Instructions" with sub-bullets when one or two sentences would do. "Setup Required" listing standard env vars the README covers. Naming choices the next session would make the same way.

When in doubt, omit. Bloat erodes trust in the snippet.

### 3. Build the snippet

Assemble the handoff using the format for the chosen mode (below). Treat the format examples as *illustrative shapes*, not templates to fill. Section names, ordering, and granularity are judgment calls — let the content drive the structure, not the other way around. The one hard requirement is the disclaimer, because it primes the next session to read the snippet critically rather than as authoritative spec.

Lean concrete over abstract. "Tried passport.js, conflicted with Express middleware (req.user always undefined), switched to oauth4webapi" is worth ten lines of "tried various auth libraries."

### 4. Self-review pass before reply

The failure mode differs per mode, so the review question does too:

- **Full** — dominant risk is template-filled bloat. Ask: *can the next session reasonably do without this section?* If yes, cut it. Also ask: *am I keeping this because it's genuinely useful, or because the format example showed it?* If the latter, cut.
- **Quick** — dominant risk is under-inclusion that breaks the next session. Ask: *is there load-bearing context I left out?* If yes, add it. *Is there padding that doesn't earn its place inside the budget?* If yes, cut it.

### 5. Reply format

Your entire reply has three parts, in order:

1. **One line above the block:** `Snip ready — copy the block below into your new session:`
2. **A code block fenced with four backticks** (` ```` `), language tag `markdown`, containing the snippet.
3. **One line below the block:** `Anything to adjust before you copy?`

The outer fence is four backticks specifically so that any 3-backtick code blocks inside the handoff (TypeScript signatures, JSON examples, shell commands) render correctly when the user pastes the snippet into their next session. This is standard CommonMark nesting.

The trailing question is an explicit invitation to revise. If the user spots something off, regenerate. If not, they copy and leave — you're done.

## Format: full

**Hard requirements** (design constraints — don't drop):

- Title line: `# Handoff: [concrete task title]`
- Disclaimer blockquote, immediately after the title

**Almost always present** (omit only with a clear reason):

- Goal — what the user is trying to accomplish.
- Next — the immediate next move.

**Common opt-in sections** — include only when they earn their place per step 2. Candidates include `## Failed Approaches`, `## Key Decisions`, `## Code Context`, `## Current State`, `## Files to Know`, `## Setup`, `## Warnings`. Section names and granularity are your call — pick what fits the actual content. Order so the most load-bearing material appears first. Use `##` headers when there are 3+ distinct chunks; inline `**Label**:` lines for shorter snippets.

**Disclaimer text** (verbatim):

> Handoff written from session memory. Verify anything load-bearing before acting.

**Worked example** — illustrative, not a template. Note how it uses only the sections that earn their place; Code Context, Files to Know, Setup, Key Decisions, and Current State were all considered and dropped because nothing load-bearing belonged there. Yours should look minimal too.

\`\`\`markdown
# Handoff: Idempotent Stripe webhook handler

> Handoff written from session memory. Verify anything load-bearing before acting.

**Branch**: feat/stripe-idempotency · **Status**: Blocked on dedupe strategy

## Goal

Make the `/webhooks/stripe` route idempotent so Stripe's automatic retries don't double-charge customers or duplicate order rows.

## Next

Pick a dedupe strategy before writing more code: a `processed_events` table (Postgres unique constraint on `stripe_event_id`) vs. Redis `SETNX` with a 24h TTL. Then wire it into `app/webhooks/stripe.ts` ahead of the existing dispatch.

## Failed Approaches

- Tried catching duplicates via a unique constraint on `orders.stripe_payment_intent_id`. Blocked legitimate updates because some events (`charge.refunded`, `charge.dispute.created`) don't map 1:1 to orders.
- Tried a Postgres advisory lock keyed on event ID. Worked, but held connections during downstream email sends and starved the pool under load.

## Warnings

- `STRIPE_WEBHOOK_SECRET` rotates weekly via Doppler. Local `.env` is last week's; tests use a fixture secret. Don't be surprised when prod-shape signatures fail locally.
\`\`\`

## Format: quick

Tighter budget — aim for ~10 lines or less. Same opt-in rule as `full`: every line earns its place.

**Hard requirements:** title line, disclaimer blockquote.
**Almost always present:** **Goal**, **Done**, **Next** as inline labels.
**Opt-in:** **Watch out**, **Failed approaches** — only when there's something real to add.

**Disclaimer text** (verbatim):

> Quick handoff — trimmed for brevity, may miss context.

**Worked example** — illustrative, not a template. Failed approaches was considered and dropped because nothing went sideways yet; Watch out earned its place because the test's I/O behavior is a real gotcha.

\`\`\`markdown
# Handoff: Flaky upload test on macOS only

> Quick handoff — trimmed for brevity, may miss context.

**Goal**: Find why `tests/upload.test.ts` passes on Linux CI but fails ~30% of local macOS runs.
**Done**: Reproduced locally; narrowed to multipart parsing, not the upload itself.
**Next**: Re-run with `DEBUG=busboy:*` and check whether boundary header parsing differs across macOS and Linux Node 20 builds.
**Watch out**: Test uses real disk I/O (no temp-fs mock) — failures may be filesystem-timing, not code.
\`\`\`
