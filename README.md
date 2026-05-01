# handoff

A Claude Code skill that produces a **copy-paste-ready handoff snippet** so a fresh Claude session can pick up where the last one left off.

No `HANDOFF.md`. No clipboard write. No temp files. Just a markdown block in Claude's reply that you copy into your next session.

## Why

Claude Code sessions get bloated or hit context limits. When you start fresh, the new session has no idea what you were doing. Most handoff tools fix this by writing a `HANDOFF.md` to disk — fine if you stay in the same repo on the same machine. Less fine if you don't.

This skill produces a single self-contained markdown snippet you can paste anywhere — same repo, different repo, different machine, web Claude, anywhere. Then you say "continue from this" and the new session has full context.

If you specifically want a `HANDOFF.md` on disk, just ask — Claude can do that with one extra prompt. The default is the snippet.

## Install

Symlink (recommended — edits to this repo are live):

```bash
ln -s /path/to/this/repo ~/.claude/skills/handoff
```

Or clone directly into the skills directory:

```bash
git clone https://github.com/ikkeseb/handoff ~/.claude/skills/handoff
```

That's it. No plugin manifest, no marketplace registration.

## Usage

In Claude Code:

```
/handoff           # full handoff (default)
/handoff full      # same as above
/handoff quick     # 5-line minimal handoff
```

Natural language works too: "create a handoff", "handoff snip", or any message containing the word "handoff".

Claude will:

1. Read the conversation and run a few `git` commands for ground truth
2. Build the snippet
3. Show it inside a 4-backtick markdown block
4. Ask if you want to adjust anything before you copy

Then in your next session, paste it and say "continue from this handoff" — or whatever phrasing you like. The format is self-describing, so any agent reading it can pick up and run.

## When the skill offers itself

If you say something like "starting a new session", "context is full", or "switching to a fresh agent", Claude will tack on a one-line offer: *"Want a handoff snip before you start fresh?"* If you ignore it, it drops. No nagging.

## Snippet format

### Full

````
# Handoff: refactor auth middleware

**Generated**: 2026-05-01 14:30
**Branch**: feature/auth
**Status**: In Progress

## Goal

Replace passport.js with oauth4webapi for OAuth2 flow.

## Completed

- [x] Removed passport dependency
- [x] Scaffolded oauth4webapi config in src/auth/

## Not Yet Done

- [ ] Implement token refresh
- [ ] Add logout endpoint

## Failed Approaches

- Tried passport.js first — conflicted with existing Express middleware. `req.user` was always undefined because passport's session middleware ran after our custom auth check. Switched to oauth4webapi which works directly with fetch.

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| oauth4webapi over passport | Lighter, no middleware conflicts |
| httpOnly cookie for refresh token | More secure than localStorage |

## Current State

**Working**: Login flow returns valid access tokens
**Broken**: Refresh endpoint returns 500 — `TokenExpiredError` at refresh.ts:42

## Files to Know

| File | Why |
|------|-----|
| `src/api/auth/refresh.ts` | Where the bug is |
| `src/auth/oauth.ts` | New OAuth client setup |

## Code Context

```typescript
// useAuth hook signature
function useAuth(): {
  user: User | null;
  login: (creds: Credentials) => Promise<void>;
  logout: () => void;
  isLoading: boolean;
}

// POST /api/auth/login response
{ "accessToken": "eyJ...", "expiresIn": 3600 }
// refresh token is set as httpOnly cookie, not in response body
```

## Resume Instructions

1. Fix `refresh.ts:42` — JWT verify uses the wrong secret
2. Add logout endpoint that clears the httpOnly cookie
3. Test full flow:
   - Login with `test@example.com` / `testpass123`
   - Wait 5 seconds, trigger refresh
   - Expected: new access token returned
   - If it fails: check `OAUTH_CLIENT_SECRET` env var

## Setup Required

- `OAUTH_CLIENT_SECRET` must be set (see `.env.example`)
- Test user: `test@example.com` / `testpass123`

## Warnings

- OAuth provider sandbox resets daily at midnight UTC
- Don't switch back to localStorage for tokens — we deliberately chose httpOnly cookies
````

Sections drop when empty, except `Failed Approaches` — that one always appears (`None — happy path so far.` if nothing failed). The explicit "None" tells the next session you considered it, instead of forgetting to include it.

### Quick

````
# Handoff: refactor auth middleware

**Goal**: replace passport.js with oauth4webapi
**Done**: removed passport, scaffolded new module
**Next**: implement token refresh in src/auth/refresh.ts
**Watch out**: don't reuse the old session middleware
````

## When to use which

- **Full** — when the next session needs to know *why* things are the way they are. Failed attempts, decisions, gotchas, partial state.
- **Quick** — for clean happy-path tasks. Refactors, mechanical edits, anything where there's nothing surprising to remember.

## What this skill won't do

- **Won't write to disk.** The snippet only exists in Claude's reply.
- **Won't auto-detect handoffs on session start.** There's no resume-side mechanism. Paste the snippet, tell the new session to continue. Markdown is self-explanatory.
- **Won't replace `git log`.** If you want commit history, use git. The skill captures session-level context that git can't see — failed approaches, design rationale, in-progress thinking.

## Acknowledgments

Snippet structure inspired by [willseltzer/claude-handoff](https://github.com/willseltzer/claude-handoff) — the file-based prior art this skill is a copy-paste alternative to.

## License

MIT
