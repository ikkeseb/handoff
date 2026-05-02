---
name: handoff
description: Generate a copy-paste-ready handoff snippet so a fresh Claude Code session can pick up where this one left off. Use when the user invokes "/handoff", "/handoff full", "/handoff quick", "create handoff", "handoff snip", or any phrase containing the word "handoff". Also offer the snippet in a single line when the user signals they're switching sessions or hitting context limits — e.g. "starting a new session", "context is full", "context bloated", "fresh context", "switching to a fresh agent". The skill writes nothing to disk; the entire handoff lives inside a 4-backtick markdown block in the reply, ready for the user to copy into their next session. Args: `quick` for a tight summary (~10 lines), `full` (default) for a longer handoff. Both modes use opt-in inclusion — sections appear only when they earn their place, never as template-fill.
---

# Handoff

Generate a copy-paste-ready snippet that lets a fresh Claude Code session pick up where this one left off — without writing any files.

## What this skill does

The user invokes `/handoff` (typically near the end of a session). You produce a single 4-backtick markdown block containing a structured handoff. The user copies it into their next session and tells that Claude to continue.

This skill writes to nothing. No `HANDOFF.md`, no clipboard, no temp file. The snippet lives in your reply and nowhere else. This is deliberate — it lets the user transfer context into a different repo, machine, or even web Claude, where on-disk handoff files don't help.

## When to use

**Direct invocation:** `/handoff`, `/handoff full`, `/handoff quick`, "create handoff", "handoff snip", or any user message containing "handoff".

**Pause signals:** when the user says they're switching sessions or hitting context limits — phrases like "starting a new session", "context is full", "context bloated", "fresh context", "switching to a fresh agent", "let me reset context". On these, offer the snippet in **one line at the end of your normal reply**: "Want a handoff snip before you start fresh?" Don't push beyond that single offer. If the user ignores it, drop it.

**Don't trigger on:** vague pause cues like "I need a break" (could mean anything), or simple milestone completions. A handoff is for session transfer, not checkpointing.

## Mode selection

Parse the invocation:

- `/handoff` or no arg → `full`
- `/handoff full` → `full`
- `/handoff quick` → `quick`
- "create handoff", "lag handoff", or similar with no qualifier → `full`
- "quick handoff" or any phrase with "quick" → `quick`

Both modes use the same opt-in inclusion rule. The difference is budget: `quick` aims for a tight ~10-line briefing; `full` runs as long as the relevant content actually needs.

## Workflow

### 1. Gather signals

Run these in parallel, tolerating failure silently. Outside a git repo, skip without comment.

- `git status --short`
- `git log --oneline -5`
- `git diff --stat`
- `git branch --show-current`

The point is to anchor your claims in ground truth instead of memory. Models often misremember which files were actually edited vs. just discussed; git output corrects that. Don't pad the snippet with these outputs verbatim — the next session can run the commands themselves.

### 2. Decide what earns its place

This is the most important step. **Do not start filling out a template.** Instead, ask for each candidate piece of content:

> *Will the next session waste time, repeat a mistake, or miss the goal without this?*

If the answer is no — omit it. Default for every section is "leave out." A section earns inclusion only when it gives the next session something they cannot trivially derive from git, the codebase, or a quick re-read.

Concrete examples of what usually **earns** its place:

- **Goal** — almost always. The next session needs to know what they're picking up.
- **Next** — almost always. The single most useful piece of content.
- **Failed approaches** — when there were any. "Tried X, failed because Y, switched to Z" saves the next session hours. Do **not** pad with "None — happy path so far"; absence is itself the signal.
- **Code context** — only when there are signatures, response shapes, or non-obvious logic the next session can't easily re-derive from the code.
- **Key decisions** — only when the rationale is non-obvious from the code or commit history.
- **Warnings / setup / current state** — only when there's something real and non-default to flag.

Concrete examples of what usually does **not** earn its place:

- A "Files to Know" table that just restates `git status --short`.
- A "Working / Broken" subsection that paraphrases what the code does.
- Numbered "Resume Instructions" with "Expected outcome / If it fails" sub-bullets, when the next step is really one or two sentences.
- "Setup Required" listing standard env vars the project's README already covers.
- "Decisions" that are just naming choices the next session would make the same way.

When in doubt, omit. Bloat erodes trust in the snippet.

### 3. Build the snippet

Assemble the handoff using the format for the chosen mode (below). Every snippet starts with the disclaimer for that mode — this primes the next session to read the snippet critically rather than as authoritative spec.

Lean concrete over abstract. "Tried passport.js, conflicted with Express middleware (req.user always undefined), switched to oauth4webapi" is worth ten lines of "tried various auth libraries."

### 4. Self-review pass before reply

Run a self-review pass on the assembled snippet. The failure mode differs per mode, so the review question does too:

- **Full** — dominant risk is template-filled bloat. Ask: *can the next session reasonably do without this section?* If yes, cut it.
- **Quick** — dominant risk is under-inclusion that breaks the next session. Ask: *is there load-bearing context I left out?* If yes, add it. *Is there padding that doesn't earn its place inside the budget?* If yes, cut it.

This pass takes seconds and is the single biggest defense against either failure mode.

### 5. Reply format

Your entire reply has three parts, in order:

1. **One line above the block:** `Snip ready — copy the block below into your new session:`
2. **A code block fenced with four backticks** (` ```` `), language tag `markdown`, containing the snippet.
3. **One line below the block:** `Anything to adjust before you copy?`

The outer fence is four backticks specifically so that any 3-backtick code blocks inside the handoff (TypeScript signatures, JSON examples, shell commands) render correctly when the user pastes the snippet into their next session. This is standard CommonMark nesting.

The trailing question is an explicit invitation to revise. If the user spots something off, regenerate. If not, they copy and leave — you're done.

## Format: full

**Spine — always present:**

- Title: `# Handoff: [concrete task title]`
- Disclaimer (blockquote, immediately after title)
- `## Goal`
- `## Next`

**Opt-in additions** — include only when they earn their place per step 2. Common candidates: `## Failed Approaches`, `## Key Decisions`, `## Code Context`, `## Current State`, `## Files to Know`, `## Setup`, `## Warnings`. Order them so the most load-bearing content appears first.

**Disclaimer text** (verbatim):

> Handoff written from session memory. Verify anything load-bearing before acting.

**Example shape** (omit any section that doesn't earn its place):

\`\`\`markdown
# Handoff: [concrete task title]

> Handoff written from session memory. Verify anything load-bearing before acting.

**Branch**: [git branch] · **Status**: [In Progress / Blocked / Ready for Review]

## Goal

[1–2 sentences. What the user wants.]

## Next

1. [Specific first action — one or two sentences]
2. [Optional second step if genuinely needed]

## Failed Approaches

- Tried X. Failed because Y. Switched to Z because [reason].

## Code Context

\`\`\`typescript
// useAuth signature
function useAuth(): {
  user: User | null;
  login: (creds: Credentials) => Promise<void>;
  logout: () => void;
}
\`\`\`

## Warnings

- [gotcha or trap that looks wrong but is intentional]
\`\`\`

## Format: quick

Tighter budget — aim for ~10 lines or less. Same opt-in rule as `full`: every line earns its place.

**Always include:** title, disclaimer, **Goal**, **Done**, **Next**.
**Opt-in:** **Watch out**, **Failed approaches** — only when there's something real to add.

**Disclaimer text** (verbatim):

> Quick handoff — trimmed for brevity, may miss context.

**Example shape:**

\`\`\`markdown
# Handoff: [task in 5 words or less]

> Quick handoff — trimmed for brevity, may miss context.

**Goal**: [one sentence]
**Done**: [comma-separated list, or "Nothing yet"]
**Next**: [the single most important next step]
**Watch out**: [one warning — omit line if nothing real]
**Failed approaches**: [one or two short notes — omit line if none]
\`\`\`
