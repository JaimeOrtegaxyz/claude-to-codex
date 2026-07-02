# claude-to-codex

Two slash commands for getting a second opinion from [Codex](https://developers.openai.com/codex/cli) without leaving Claude Code. Claude briefs Codex on the live session first, *then* asks — so Codex sees the conversation you're actually having, not just your one-liner and the repo.

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
