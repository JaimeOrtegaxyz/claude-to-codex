# claude-to-codex

I kept wanting a second opinion from [Codex](https://developers.openai.com/codex/cli) without leaving Claude Code. The naive version of that — pipe my prompt straight to `codex exec` — turned out useless: Codex only ever saw my one-liner plus the repo on disk, never the conversation I was actually having, so it kept confidently suggesting things we'd already ruled out. These two slash commands fix that: Claude briefs Codex on the live session first, *then* asks.

## What's here

| Command | What it does |
|---|---|
| `/codex <question>` | Consults Codex **with a context briefing** Claude assembles from the current conversation, and **keeps memory across turns** within a Claude session (each Claude session gets its own isolated Codex thread; auto-falls back to a fresh session if the stored id expired). |
| `/codex-btw <question>` | One-shot quick aside with a one-line context — **no session, no memory**, doesn't touch the `/codex` thread. Analogous to Claude Code's `/btw`. |

Both run `codex exec` against the repo in your current working directory, so Codex also sees the code.

## Install

Requires the [`codex` CLI](https://developers.openai.com/codex/cli) on your `PATH` and logged in (`codex login`).

Copy the two command files into your Claude Code commands directory:

```bash
git clone https://github.com/JaimeOrtegaxyz/claude-to-codex.git
cp claude-to-codex/codex.md claude-to-codex/codex-btw.md ~/.claude/commands/
```

They'll show up as `/codex` and `/codex-btw` in Claude Code. That's it.

## Versions

- **v2.0.0** — Commands now brief Codex with conversation context before asking (the whole point above). `/codex-btw` gains a one-line context field. Session/memory mechanics unchanged.
- **v1.0.0** — Initial release: transparent passthrough of the prompt to `codex exec`.

MIT.
