---
description: One-shot quick aside to Codex — no session, no memory, doesn't affect /codex thread
allowed-tools: Bash(codex:*), Bash(cat:*)
---

Ask Codex a quick standalone question. Does NOT use or extend the per-session `/codex` thread — analogous to Claude Code's `/btw`.

User prompt: $ARGUMENTS

Run this single bash block and return ONLY the contents of `$LAST`. No commentary unless asked.

```bash
set -u
LAST="/tmp/codex-btw-last.txt"
FULL="/tmp/codex-btw-full.txt"
PROMPT=$(cat <<'CODEX_EOF'
$ARGUMENTS
CODEX_EOF
)

printf '%s' "$PROMPT" | codex exec \
  --skip-git-repo-check --color never \
  -C "$PWD" -o "$LAST" - \
  > "$FULL" 2>&1

cat "$LAST"
```

If `$LAST` is empty, fall back to surfacing the tail of `$FULL` so the error is visible.
