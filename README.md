# handoff

A Claude Code skill that produces a **copy-paste-ready handoff snippet** so a fresh Claude session can pick up where the last one left off.

No `HANDOFF.md`. No clipboard write. No temp files. Just a markdown block in Claude's reply that you copy into your next session — same repo, different repo, different machine, web Claude, anywhere.

If you specifically want a `HANDOFF.md` on disk, ask — Claude will do it with one extra prompt. The default is the snippet.

## Install

```bash
git clone https://github.com/ikkeseb/handoff ~/.claude/skills/handoff
```

Or symlink your own clone (edits go live):

```bash
ln -s /path/to/your/clone ~/.claude/skills/handoff
```

## Usage

```
/handoff           # full handoff (default)
/handoff quick     # tight ~10-line summary
```

Claude reads the conversation, optionally checks `git` for ground truth, and shows the snippet inside a 4-backtick markdown block. Paste it into your next session and say "continue from this".

## Snippet structure

Both modes use **opt-in inclusion** — sections appear only when they give the next session something they can't trivially get from git or the codebase. Default is to omit. A snippet that earns every line beats one that follows a template.

**Full** — minimum spine of Goal + Next, plus any of Failed Approaches, Code Context, Key Decisions, Current State, Warnings, etc. when relevant. As long as it needs to be, no longer.

**Quick** — tighter budget (~10 lines). Goal + Done + Next always; Watch out and Failed approaches when there's something real to add.

Both snippets lead with a disclaimer — the snippet is generated from session memory, may be incomplete or wrong, and the next session should verify load-bearing claims against git/code rather than treat it as spec.

## What this skill won't do

- **Won't write to disk.** Snippet only exists in Claude's reply.
- **Won't auto-detect on session start.** Paste the snippet, tell the new session to continue. Markdown is self-explanatory.
- **Won't replace `git log`.** Captures session-level context git can't see — failed approaches, design rationale, in-progress thinking.

## License

MIT
