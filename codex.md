---
description: Consult Codex with full session context + persistent memory across turns within this Claude session
allowed-tools: Bash(codex:*), Bash(cat:*), Bash(grep:*), Bash(awk:*), Bash(rm:*), Bash(mkdir:*), Bash(rmdir:*), Bash(sleep:*), Bash(test:*)
---

Consult Codex about: **$ARGUMENTS**

Codex `exec` runs against the repo in `$PWD`, and within this Claude session it remembers prior `/codex` turns. But it has **none of our actual conversation** — not the errors we've hit, what we've already tried and ruled out, what we discovered at runtime, or the decisions we've made. Handed only the bare prompt plus the repo, it reasons from code alone and hands back advice we've already disproven. So brief it before you ask.

## Step 1 — Build the briefing

Replace the `CONTEXT_BRIEF` placeholder line in the bash block below with a tight, self-contained briefing drawn from **this** conversation. Include what Codex needs and can't see for itself:

- **Goal** — what we're actually trying to do right now.
- **Symptoms / errors** — paste the real error text, status codes, or output, trimmed to what matters.
- **Already tried / ruled out** — so it doesn't re-suggest dead ends.
- **Findings & decisions** — anything learned at runtime or settled on (e.g. "endpoint X returns 403 since the Feb 2026 change", "we chose approach Y over Z").
- **Relevant files / paths / symbols** — point it at the code in `$PWD` that matters.

Rules:
- Complete but lean — the facts Codex needs, not a transcript.
- **Never** paste secrets (tokens, client secrets, cookies, API keys, passwords). Redact them.
- On follow-up `/codex` turns, be briefer about things covered in earlier turns (Codex remembers those) but always include what's **new** since the last turn.
- If there genuinely is no prior context, write one line saying so and state the standalone question.

## Step 2 — Run it

Run this single bash block. It sends `briefing + question` to Codex, continuing this session's Codex thread (and falls back to a fresh session if the stored id expired).

```bash
set -u
SID="${CLAUDE_SESSION_ID:-$PPID}"
STATE="/tmp/codex-session-$SID"
LOCK="/tmp/codex-session-$SID.lock.d"
LAST="/tmp/codex-last-$SID.txt"
FULL="/tmp/codex-full-$SID.txt"

CONTEXT=$(cat <<'CTX_EOF'
CONTEXT_BRIEF — replace this entire line with the briefing from Step 1.
CTX_EOF
)
QUESTION=$(cat <<'CODEX_EOF'
$ARGUMENTS
CODEX_EOF
)
PROMPT="You're being consulted by Claude Code, which is pair-working with me in a live session you can't see. Here is the context it has that you don't:

=== SESSION CONTEXT ===
$CONTEXT
=== END CONTEXT ===

What we need from you:
$QUESTION"

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

## Step 3 — Return the reply

- Return the contents of `$LAST` — Codex's clean final reply — to me.
- If `$LAST` is empty, read `$FULL` and surface whatever error Codex emitted.
- Codex answered off your briefing, so if its advice contradicts something we already know to be true (e.g. it didn't account for a runtime finding), add a one-line flag after its reply — but don't suppress or rewrite what it said.
- To reset the thread (fresh Codex session next call), `rm -f /tmp/codex-session-$SID` first.
