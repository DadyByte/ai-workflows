# Examples

Two real files from my Claude Code setup, with personal paths, names and ids replaced by placeholders.

- [agent-os-loop.md](agent-os-loop.md) is the global rule file that loads into every session. It holds the seven-step loop: read before deriving, write back in the same session, end with a gate. The case study is [01-session-loop.md](../workflows/01-session-loop.md).
- [hooks.settings.json](hooks.settings.json) is an abridged excerpt of the `hooks` block of a Claude Code `settings.json`. It shows the three hooks that carry the loop: a project line and nudges at session start, a recall hook on every prompt, and a marker line at session end. The real file also has a PreToolUse hook that compresses CLI output. `$VAULT` stands for the knowledge base folder.

Every hook command ends in `2>/dev/null || true`, so it fails open on purpose. A retrieval aid must never be the reason a prompt or a session fails. The cost is that a silent hook is ambiguous: no match and a broken hook look the same, and nothing monitors for a dead hook.

The scripts these hooks call are not published.

The rule file's "7 of 13" figure (2026-09-14) is unverified. It was flagged on 2026-09-28 because nobody had checked whether automation transcripts were excluded from that count.
