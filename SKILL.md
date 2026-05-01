---
name: handoff
description: Generate a copy-paste-ready handoff snippet so a fresh Claude Code session can pick up where this one left off. Use when the user invokes "/handoff", "/handoff full", "/handoff quick", "create handoff", "handoff snip", or any phrase containing the word "handoff". Also offer the snippet in a single line when the user signals they're switching sessions or hitting context limits — e.g. "starting a new session", "context is full", "context bloated", "fresh context", "switching to a fresh agent". The skill writes nothing to disk; the entire handoff lives inside a 4-backtick markdown block in the reply, ready for the user to copy into their next session. Args: `quick` for a 5-line summary, `full` (default) for the complete handoff with goal, decisions, failed approaches, code context, and resume instructions.
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

`full` is the default because that's the typical need. `quick` is a deliberate ask.

## Workflow

### 1. Gather signals

Run these in parallel, tolerating failure silently. Outside a git repo, skip without comment.

- `git status --short`
- `git log --oneline -5`
- `git diff --stat`
- `git branch --show-current`

The point is to anchor "Completed" and "Current State" to ground truth instead of memory. Models often misremember which files were actually edited vs. just discussed; git output corrects that.

### 2. Build the handoff from the conversation

Read the session and pull out:

- **Goal** — what the user actually wants. Their first substantive message, refined by any corrections they made along the way.
- **Completed** — what's actually done. Cross-check against `git status` / `git log`. Don't list intentions as completions.
- **Not yet done** — concrete remaining tasks, not vague aspirations.
- **Failed approaches** — what was tried and abandoned, and *why*. This is the highest-value part: it stops the next session from repeating the same mistakes.
- **Key decisions** — choices made and their rationale. "We used X because Y" — never just "we used X."
- **Current state** — what works, what's broken (with verbatim error messages where available).
- **Code context** — concrete signatures, response shapes, non-obvious logic the next session will need. Show, don't describe.
- **Resume instructions** — specific next steps with expected outcomes.
- **Setup, warnings** — only if there are real ones.

Lean concrete over abstract. "Tried passport.js, conflicted with Express middleware (req.user always undefined), switched to oauth4webapi" is worth ten lines of "tried various auth libraries."

### 3. Reply format

Your entire reply has three parts, in order:

1. **One line above the block:** `Snip ready — copy the block below into your new session:`
2. **A code block fenced with four backticks** (` ```` `), language tag `markdown`, containing the full or quick handoff content (per the format sections below).
3. **One line below the block:** `Anything to adjust before you copy?`

The outer fence is four backticks specifically so that any 3-backtick code blocks inside the handoff (TypeScript signatures, JSON examples, shell commands) render correctly when the user pastes the snippet into their next session. This is standard CommonMark nesting.

The trailing question is an explicit invitation to revise. If the user spots something off, regenerate. If not, they copy and leave — you're done.

## Format: full

Use this structure. **Drop sections that are empty, EXCEPT `Failed Approaches` — that section always appears, with `None — happy path so far.` if nothing failed.** The explicit "None" tells the next session that you considered it, rather than forgot to include it.

```markdown
# Handoff: [concrete task title, not vague]

**Generated**: [YYYY-MM-DD HH:MM]
**Branch**: [git branch, or "no git" if outside a repo]
**Status**: [In Progress / Blocked / Ready for Review]

## Goal

[1–2 sentences. What the user wants.]

## Completed

- [x] [specific item]
- [x] [another]

## Not Yet Done

- [ ] [specific remaining task]
- [ ] [another]

## Failed Approaches

- Tried X. Failed because Y. Switched to Z because [reason].
- ...

(or: "None — happy path so far.")

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| [choice] | [why] |

## Current State

**Working**: [what functions right now]
**Broken**: [what doesn't, with error messages verbatim]

## Files to Know

| File | Why |
|------|-----|
| `path/to/file.ts` | [brief description] |

## Code Context

[Concrete artifacts only. Signatures, response shapes, non-obvious logic. Don't describe — show.]

\`\`\`typescript
// useAuth signature
function useAuth(): {
  user: User | null;
  login: (creds: Credentials) => Promise<void>;
  logout: () => void;
}
\`\`\`

## Resume Instructions

1. [Specific first action with command or file edit]
2. [Verification step with expected outcome]
   - Expected: [what should happen]
   - If it fails: [what to check]
3. ...

## Setup Required

- Env vars: `API_KEY`, `DATABASE_URL`
- Test accounts: test@example.com / password123
- Required services: Redis on :6379

## Warnings

- [gotcha or trap that looks wrong but is intentional]
```

## Format: quick

Exactly these five lines. No extras. If you find yourself wanting to add a sixth field, the user should have asked for `full`.

```markdown
# Handoff: [task in 5 words or less]

**Goal**: [one sentence]
**Done**: [comma-separated list, or "Nothing yet"]
**Next**: [the single most important next step]
**Watch out**: [one warning, or "Nothing special"]
```

## Principles

- **Failed approaches are gold.** "Tried X, didn't work because Y" saves the next session hours. This is willseltzer's insight and it's right: when in doubt, include them.
- **Show code, don't describe.** "Created a hook" is useless. "Hook signature: `useAuth(): { user, login, logout, isLoading }`" is useful.
- **Verification needs expected outcomes.** "Test it" is hand-waving. "POST /api/login with `test@example.com` / `testpass`, expect 200 with `{ token }`" is a real test step.
- **Concrete over abstract.** Use file paths, function names, error messages verbatim, real test data.
- **Brutally concise.** Every word should earn its place. The next session will read this in full — make it worth reading.
