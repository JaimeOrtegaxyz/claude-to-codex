# Contributing to claude-to-codex

Thanks for your interest! This repo is intentionally tiny — two Markdown slash-command files — so contributing is simple.

## Code of Conduct

This project follows our [Code of Conduct](CODE_OF_CONDUCT.md). By participating you agree to uphold it.

## Trying your change

There's no build or test suite. To test a change, copy the edited command file into your own Claude Code commands directory and use it:

```bash
cp codex.md codex-btw.md ~/.claude/commands/
```

Then run `/codex` or `/codex-btw` in Claude Code (requires the [`codex` CLI](https://developers.openai.com/codex/cli) installed and logged in).

## Proposing a change

1. **Open an issue first** for anything non-trivial, so we can agree on the approach.
2. Fork, branch, and keep each pull request to one focused change.
3. In the PR description, say what you tested — e.g. which command you ran and what Codex received.
4. Open the PR against `main`.
