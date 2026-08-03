---
name: subagents
description: Delegate work through the subagents CLI. Use for background workers, specialized research, long-running tasks, parallel delegation, dependent multi-step work, and scripted TypeScript agent workflows.
---

# Subagents

Use `subagents` for delegated agent work. Do not launch ad-hoc `pi` sessions directly.

Choose the smallest suitable mode:

- Use `subagents create` for one independent background task.
- Use a TypeScript workflow for dependent steps, parallel branches, or reusable scripted pipelines.

## Background job

Start a worker with a standard profile:

```bash
subagents create \
  --thread "<current thread id>" \
  --instructions "$(cat /app/subagents/profiles/worker.md)" \
  --prompt "Describe the task and required artifact"
```

Use `/app/subagents/profiles/researcher.md` for focused discovery. Add `--model "provider/model:reasoning"` only when the task needs a specific model.

Manage jobs:

```bash
subagents list
subagents show <job_id>
subagents append <job_id> --prompt "Follow-up instruction"
subagents cancel <job_id>
```

After creation, keep the parent responsive. Report the job id and use `show` when status is needed. Normal background jobs notify the parent when they finish.

## TypeScript workflow

Write a workflow file anywhere. Use ordinary TypeScript control flow with Claude-style globals:

```ts
export const meta = {
  name: "review-files",
  description: "Plan and review files",
};

const plan = await agent<{ files: string[] }>(`Find files for ${args.task}`, {
  label: "plan",
  schema: {
    type: "object",
    properties: { files: { type: "array", items: { type: "string" } } },
    required: ["files"],
  },
});

const reviews = await pipeline(plan.files, (file) =>
  agent(`Review ${file}`, { label: file }),
);

return reviews.join("\n\n");
```

Available globals:

- `args`: JSON passed by `--args`; defaults to `{}`.
- `agent(prompt, options)`: run one durable agent step.
- `pipeline(items, callback)`: run mapped agent branches in parallel.
- `parallel(...promises)`: await independent branches in parallel.

`agent` options are `instructions`, `label`, `model`, and `schema`. Sequential `await`, conditions, loops, variables, and returned JSON work as normal TypeScript.

Run and inspect workflows:

```bash
subagents workflow run /absolute/path/workflow.ts \
  --thread "<current thread id>" \
  --args '{"task":"..."}'

subagents workflow list
subagents workflow show <workflow_run_id>
subagents workflow cancel <workflow_run_id>
```

The run command returns immediately. Poll `workflow show` for `completed`, `failed`, or `canceled`; the final value is in `output`. Workflow internals do not emit per-step chat notifications.

Workflow state and completed agent steps are durable in SQLite. The workflow runs from the source snapshot stored at launch, so later edits or deletion of the original file do not alter the active run.

## Generation limit

`SUBAGENTS_DEPTH` is propagated automatically. `SUBAGENTS_MAX_DEPTH` defaults to `4`; generation 4 cannot create another job or workflow. If the CLI reports the depth-limit error, return the work to a parent agent or ask an operator to raise the container limit. Do not retry the same forbidden call.
