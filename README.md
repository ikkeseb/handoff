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
/handoff quick     # 5-line minimal version
```

Natural language works too — any message containing "handoff" triggers it. If you say something like "starting a new session" or "context is full", Claude offers a snippet without nagging.

Claude reads the conversation, runs a few `git` commands for ground truth, and shows the snippet inside a 4-backtick markdown block. Paste it into your next session and say "continue from this".

## Snippet structure

**Full** — title, generated timestamp, branch, status, then sections that drop when empty: Goal, Completed, Not Yet Done, Failed Approaches, Key Decisions, Current State, Files to Know, Code Context, Resume Instructions, Setup Required, Warnings.

`Failed Approaches` always appears, even if it's just `None — happy path so far.` — explicit "None" tells the next session you considered it.

**Quick** — four lines: Goal, Done, Next, Watch out. For clean happy-path tasks where there's nothing surprising to remember.

## What this skill won't do

- **Won't write to disk.** Snippet only exists in Claude's reply.
- **Won't auto-detect on session start.** Paste the snippet, tell the new session to continue. Markdown is self-explanatory.
- **Won't replace `git log`.** Captures session-level context git can't see — failed approaches, design rationale, in-progress thinking.

## License

MIT
