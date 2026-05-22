---
name: agent-system-orchestration
description: Use when a task should be delegated to researcher or worker child agents in this persistent multi-agent system.
---

# Agent System Orchestration

Use this skill when a user request is heavy, research-oriented, implementation-oriented, or naturally parallelizable.

The orchestrator coordinates child agents through bash. Use the current turn's `Thread id` and `Child agent command` values.

## Researcher

Run a one-shot researcher when a task needs focused discovery, context gathering, source reading, or fact collection.

```bash
<child-agent-command> research --thread <thread-id> --task "..."
```

The command prints a JSON object with an artifact path. Read that artifact before planning worker execution or replying.

## Worker

Start a persistent worker when a task needs implementation, editing, verification, or iterative follow-up.

```bash
<child-agent-command> worker start --thread <thread-id> --task "..." --context "..."
```

The command prints a JSON object with `workerId` and an artifact path. Keep the worker id when the same task should be continued.

Continue an existing worker:

```bash
<child-agent-command> worker send --thread <thread-id> --worker "$WORKER_ID" --message "..."
```

List known workers for the current thread:

```bash
<child-agent-command> workers list --thread <thread-id>
```

## Parallel Work

Use shell background jobs and `wait` when independent researchers or workers can run concurrently.

Read child-agent artifacts before producing the final user-facing reply.
