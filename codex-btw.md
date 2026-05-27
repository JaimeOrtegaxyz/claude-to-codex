---
description: One-shot quick aside to Codex with light context — no session, no memory, doesn't affect /codex thread
allowed-tools: Bash(codex:*), Bash(cat:*)
---

Ask Codex a quick standalone question. Does NOT use or extend the per-session `/codex` thread — analogous to Claude Code's `/btw`.

Consult Codex about: **$ARGUMENTS**

Because there's no session memory here, Codex sees only what you hand it plus the repo in `$PWD`. Give it a short context line so it isn't reasoning blind.

## Step 1 — One-line context

Replace the `CONTEXT_BRIEF` placeholder below with a one- or two-sentence briefing (goal + the key fact/error that makes the question make sense), or the literal word `none` if it's a genuinely standalone question. Never paste secrets — redact them.

## Step 2 — Run it

```bash
set -u
LAST="/tmp/codex-btw-last.txt"
FULL="/tmp/codex-btw-full.txt"
CONTEXT=$(cat <<'CTX_EOF'
CONTEXT_BRIEF — replace this entire line with the one-line context, or "none".
CTX_EOF
)
QUESTION=$(cat <<'CODEX_EOF'
$ARGUMENTS
CODEX_EOF
)
PROMPT="Quick consult from Claude Code (a live session you can't see).

Context: $CONTEXT

Question: $QUESTION"

printf '%s' "$PROMPT" | codex exec \
  --skip-git-repo-check --color never \
  -C "$PWD" -o "$LAST" - \
  > "$FULL" 2>&1

cat "$LAST"
```

Return the contents of `$LAST`. If `$LAST` is empty, fall back to surfacing the tail of `$FULL` so the error is visible.
