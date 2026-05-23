---
name: agent-system-orchestration
description: Use when a task should be delegated to ad-hoc researcher or worker Pi sessions through bash.
---

# Agent System Orchestration

Use this skill when a user request is heavy, research-oriented, implementation-oriented, or naturally parallelizable.

Researcher and worker are skill-level conventions, not built-in platform roles. Start them as normal Pi sessions from bash. Choose the model and thinking level yourself for the task.

Use `pi --list-models` when you need to inspect available model ids.

Useful Pi flags:

- `--print` / `-p`: run non-interactively and print the final answer.
- `--model <provider/model:thinking>`: choose the model and thinking level.
- `--session-dir <dir>`: choose where the raw session is stored.
- `--append-system-prompt <file>`: attach a role prompt file.
- `--tools read,bash,edit,write`: keep the usual workstation tool set.
- `--no-extensions --no-skills`: keep the child session explicit and avoid hidden extension or skill metadata injection.

Store child sessions under `raw/sessions/<thread-key>/researcher` or `raw/sessions/<thread-key>/worker`. Store child outputs as Markdown artifacts in memory, then read those artifacts before replying.

## Researcher

Run a one-shot researcher when a task needs focused discovery, context gathering, source reading, or fact collection.

```bash
THREAD_ID="<thread-id>"
THREAD_KEY="$(printf '%s' "$THREAD_ID" | tr -c 'A-Za-z0-9._-' '-')"
RUN_ID="$(date -u +%Y%m%dT%H%M%SZ)-research"
SESSION_DIR="$MEMORY_DIR/main/raw/sessions/$THREAD_KEY/researcher/$RUN_ID"
ARTIFACT="$MEMORY_DIR/main/wiki/artifacts/$THREAD_KEY/research-$RUN_ID.md"
MODEL="openai-codex/gpt-5.4-mini:medium"

mkdir -p "$SESSION_DIR" "$(dirname "$ARTIFACT")"

pi --print \
  --no-extensions \
  --no-skills \
  --tools read,bash,edit,write \
  --session-dir "$SESSION_DIR" \
  --model "$MODEL" \
  --append-system-prompt "$MEMORY_DIR/main/skills/agent-system-orchestration/researcher.md" \
  "Research this task and write a compact artifact. Memory workspace: $MEMORY_DIR. Task: ..." \
  > "$ARTIFACT"
```

Read the artifact before planning worker execution or replying.

## Worker

Start a persistent worker when a task needs implementation, editing, verification, or iterative follow-up.

```bash
THREAD_ID="<thread-id>"
THREAD_KEY="$(printf '%s' "$THREAD_ID" | tr -c 'A-Za-z0-9._-' '-')"
WORKER_ID="<stable-worker-id>"
SESSION_DIR="$MEMORY_DIR/main/raw/sessions/$THREAD_KEY/worker/$WORKER_ID"
ARTIFACT="$MEMORY_DIR/main/wiki/artifacts/$THREAD_KEY/worker-$WORKER_ID.md"
MODEL="openai-codex/gpt-5.4:high"

mkdir -p "$SESSION_DIR" "$(dirname "$ARTIFACT")"

pi --print \
  --no-extensions \
  --no-skills \
  --tools read,bash,edit,write \
  --session-dir "$SESSION_DIR" \
  --model "$MODEL" \
  --append-system-prompt "$MEMORY_DIR/main/skills/agent-system-orchestration/worker.md" \
  "Work on this task and update the artifact. Artifact path: $ARTIFACT. Task: ..." \
  > "$ARTIFACT"
```

Continue an existing worker:

```bash
pi --print \
  --no-extensions \
  --no-skills \
  --tools read,bash,edit,write \
  --session-dir "$SESSION_DIR" \
  --model "$MODEL" \
  --append-system-prompt "$MEMORY_DIR/main/skills/agent-system-orchestration/worker.md" \
  "Continue. Artifact path: $ARTIFACT. Message: ..." \
  > "$ARTIFACT"
```

List known workers for the current thread:

```bash
find "$MEMORY_DIR/main/raw/sessions/$THREAD_KEY/worker" -mindepth 1 -maxdepth 1 -type d
```

## Parallel Work

Use shell background jobs and `wait` when independent researchers or workers can run concurrently.

Read child-agent artifacts before producing the final user-facing reply.
