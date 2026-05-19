---
description: Consult Codex with persistent memory across turns within this Claude session
allowed-tools: Bash(codex:*), Bash(cat:*), Bash(grep:*), Bash(awk:*), Bash(rm:*), Bash(mkdir:*), Bash(rmdir:*), Bash(sleep:*), Bash(test:*)
---

Ask Codex a question, continuing the per-Claude-session Codex thread so it remembers prior `/codex` turns. Different Claude sessions get isolated Codex sessions. If the stored session id is invalid (expired / not found), automatically fall back to starting a fresh Codex session.

User prompt: $ARGUMENTS

Run this single bash block and return ONLY the contents of `$LAST` (Codex's clean final reply). Do not add commentary unless I asked for interpretation.

```bash
set -u
SID="${CLAUDE_SESSION_ID:-$PPID}"
STATE="/tmp/codex-session-$SID"
LOCK="/tmp/codex-session-$SID.lock.d"
LAST="/tmp/codex-last-$SID.txt"
FULL="/tmp/codex-full-$SID.txt"
PROMPT=$(cat <<'CODEX_EOF'
$ARGUMENTS
CODEX_EOF
)

# Serialize concurrent /codex calls in this session (portable, no flock)
for _ in $(seq 1 150); do
  mkdir "$LOCK" 2>/dev/null && break
  sleep 0.2
done
trap 'rmdir "$LOCK" 2>/dev/null' EXIT

run_fresh() {
  printf '%s' "$PROMPT" | codex exec \
    --skip-git-repo-check --color never \
    -C "$PWD" -o "$LAST" - \
    > "$FULL" 2>&1
  rc=$?
  grep -m1 'session id:' "$FULL" | awk '{print $3}' > "$STATE"
  return $rc
}

if [ -s "$STATE" ]; then
  CODEX_SID=$(cat "$STATE")
  : > "$LAST"
  if ! printf '%s' "$PROMPT" | codex exec resume \
      --skip-git-repo-check \
      -o "$LAST" "$CODEX_SID" - \
      > "$FULL" 2>&1; then
    rm -f "$STATE"
    run_fresh
  fi
else
  run_fresh
fi

cat "$LAST"
```

Notes for you (Claude) when handling the response:
- Output of `$LAST` is Codex's final clean reply — return it as-is.
- If `$LAST` is empty, read `$FULL` to surface whatever error Codex emitted.
- To reset the thread (start a fresh Codex session next call), `rm -f /tmp/codex-session-$CLAUDE_SESSION_ID` first.
