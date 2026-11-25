---
name: lynko-run
description: >
  Execute commands on a target machine through Lynko — builds, scripts, one-off
  inspections, smoke tests. Use when you need to run something on a specific
  host (a preconfigured dev VM, build server, or test target) and capture the
  output for review. Requires a runner node attached to the pod and at least
  one of the owner's machines ticked for the pod on its Runner page.
---

# Run Commands with Lynko

For full syntax details, see the [DSL cheatsheet](../../reference/dsl-cheatsheet.md#running-commands).

## When to use `run()` vs `test()`

- **`test()`** — runs your project's CI suite. Targets come from the CI config (Makefile, GitHub Actions, etc.). Use when you want the canonical test set.
- **`run()`** — runs any shell command on a target machine. Use for builds, scripts, inspections, smoke tests, or anything that isn't a CI target.

Both run against your current drafts on writable collections (see [Checkpoint Behavior](#checkpoint-behavior)).

## Discovery

Use `artifacts()` to discover the Runner alias attached to this pod, then check what's available:

```
runner.status()                              # This pod's machines + active runs
runner.status(machine="eval-a")             # Selected runs and labelled installed-agent inventory
runner.history()                             # Recent runs + state and duration (newest first)
```

If Runner is not attached, add it to the pod in the dashboard by reference name. Machines belong to the pod's owner, who adds them in the dashboard's Compute panel; each pod uses only the machines ticked on its Runner page (pod → Runner → **Manage**). Attaching Runner ticks the owner's machines at that moment; a machine added later reaches no pod until it is ticked. `runner.status()` lists only this pod's machines, and a machine that is not ticked is refused exactly like a name that does not exist. With several machines, select one explicitly for dispatch and installed-agent inventory. An unavailable status is a refusal, not proof of an empty configuration.

**Machine requirements.** A machine needs SSH access with a known host key, and `git` and `tmux` 3.3a or newer on the SSH account's `PATH`. Runs execute as that account, so it should be a dedicated, unprivileged account: no sudo, not in the `docker` group.

**Assume nothing is preinstalled.** Machines are typically minimal Linux (Ubuntu) by default — `bash`, `wget`, `apt` (without sudo), `python3`, and standard userland are usually available, but `node`, `curl`, language toolchains, and other dev tools are not. Probe with `which <tool>` before relying on anything beyond standard userland. To install a CLI without sudo, prefer downloading a static binary into `$HOME` and running it directly (e.g., `wget` a release zip, `unzip`, run from `$HOME/.local/bin/`).

## Dispatch and Poll

`run()` returns immediately with a run ID. Use the runner surface for state and the run selector for output:

```
my-project.run("go build ./...")
# → Run dispatched: run_7c9e4f21a3b8d05612ef9a34c1d2b7e0

runner.status()                              # Configured machines + active runs
runner.history(limit=10)                    # Recent runs + state and duration
runner["run_7c9e4f21a3b8d05612ef9a34c1d2b7e0"].read()   # Captured output
```

Terminal states in `runner.history()` are `succeeded`, `failed`, `timeout` and `canceled`. A listed `running` run is still active.

`run()` is dispatched from a collection (`my-project.run(...)`). The collection is the dispatch context; `runner.status()` and `runner.history()` report execution state, while `runner["run-ID"]` exposes output and cancellation.

**LLM-client polling pattern.** Most LLM clients time out tool calls sooner than a typical build finishes. Dispatch first, then check `runner.history()` in a separate call after a reasonable interval. Do not busy-poll or try to block inside the dispatch call.

## Reading Output

Navigate output top-down — prefer `grep` and `lines` over full `read()` for long output:

```
runner["run-ID"].read()                      # Full captured output
runner["run-ID"].lines("1-50")               # First 50 lines (use lines() for ranges, not read())
runner["run-ID"].lines("100-120")            # Specific line range
runner["run-ID"].grep("ERROR", context_lines=3)  # Find errors with context
```

During execution, `read()` / `grep()` / `lines()` query the target machine live over SSH. Once the run completes, output is captured to your pod and these operations read from the database — they keep working even if the machine is offline.

## Scoping and Selection

Scope the working directory with bracket notation:

```
my-project[src/].run("go build ./...")       # Run inside src/
my-project[docs/].run("grep -r TODO .")      # Scoped grep
```

Pick a machine when several are ticked for the pod:

```
my-project.run("./deploy.sh", machine="prod")
my-project.run("./bench.sh", machine="perf-box", timeout="30m")
```

An explicit `run()` timeout is a Go duration of a whole number of seconds, at least `1s`, including composites such as `1h30m`; `"500ms"` or `"1.5s"` is refused. It is clamped to the Runner's maximum. Without one, the machine's default timeout applies (60 minutes unless the owner set another). `invoke()` has its own grammar in the [DSL cheatsheet](../../reference/dsl-cheatsheet.md#invoking-agents).

A dispatched run keeps its machine snapshot: editing, deleting or unticking the machine never strands it, and `runner["run-ID"]` still reads and cancels it. Renames or coordinate changes can select a different/cold working copy; preserve authored work in all affected copies before owner-side edits.

## Slots and Working-Copy Selection

A run of one collection on a machine takes one of the pod's slots there: the machine's `slots` (1–4, default 4, set by the owner in Compute) is how many runs of one collection a pod can have on that machine at once; more wait their turn. A new slot may need a cold clone; a previously prepared slot is reused warm when safe. Other pods have independent slots even when they reference the same Git collection on the same machine.

When several working copies have been prepared, `run.status()` lists the available slots, while an unqualified `run.capture()` refuses ambiguity rather than guessing. Select the copy by the **run ID that last prepared it**:

```
my-project.run.status(machine="dev", run="run_0123456789abcdef0123456789abcdef")
my-project.run.capture(machine="dev", run="run_0123456789abcdef0123456789abcdef", include_untracked=true)
```

Use the actual run ID returned by `run()`, not the illustrative value above. Capture is refused when a command is active in that selected slot; a command in a different slot does not block it. A superseded run ID cannot select a newer preparation. A working copy on a machine that is no longer ticked for the pod is refused.

## Cancelling

```
runner["run-ID"].cancel()
```

Cancel is cooperative — the current command is interrupted and the run is marked `canceled`. Output captured before cancel is preserved.

## Typical Workflow

```
# 1. Edit some code
my-project[src/server.go].expand("Handler").draft.edit("old", "new")

# 2. Dispatch a build/smoke check against drafts
my-project.run("go build ./... && ./bin/smoke-test")

# 3. Check state after a reasonable interval
runner.history(limit=10)

# 4. When done — review output
runner["run-ID"].grep("FAIL|ERROR", context_lines=3)   # Look for problems
runner["run-ID"].lines("1-30")                         # Check the start

# 5. Commit if it worked
my-project.commit("feat(handler): ...")
```

## Checkpoint Behavior

`run()` records its exact source before dispatching and executes it without publishing it:

- **No effective staged changes** — the run uses the exact tracked commit recorded for it, on both writable and read-only remote collections. It does not follow later advances of the remote branch.
- **Staged changes** — the run uses source prepared from the tracked commit and your current staged changes. Your drafts stay staged: `run()` does not commit or discard them, and it does not push a checkpoint to a remote system branch.

This also works when a previously pushable remote now refuses pushes, while the collection remains recorded as supporting pushes. Creating new drafts on a collection without recorded push support remains unavailable.

## Tips

- Concurrent collection runs: each pod has its own slots per `(collection, machine)` (default 4; owner-configurable 1–4). Several runs can execute at once in separate slots; when all slots are busy, later runs wait their turn and fail `WORKSPACE_HELD` if no slot comes free within their bound. At 16 pending or running runs of one collection on one machine, `run()` and `invoke()` refuse with `RUN_IN_PROGRESS`. Use `runner.status()` to inspect holders and waiters.
- Long-running commands survive client disconnects. The run continues on the machine (via tmux); `runner.history()` reports its final state and duration.
- `runner.history()` paginates: when more runs exist it prints the next-page call, such as `runner.history(cursor="…")`. Run it as printed; following those calls shows each run exactly once.
- Captured output is bounded at ~10MB (head+tail split past the limit, with a `--- OUTPUT TRUNCATED ---` marker between the halves). `read()`, `grep()`, and `lines()` all operate on the captured output; if truncation loses something you need, scope the command to produce less output or redirect to a file on the target and read it with a follow-up run.
- For multi-line scripts with complex escapes (Python heredocs, embedded quotes, `\n` in strings), prefer writing the script to `/tmp/` first and executing by path, rather than passing the script inline. Pattern: `cat > /tmp/foo.py << 'EOF' ... EOF && python3 /tmp/foo.py`. Inline heredocs occasionally trip the output-capture path on short-lived commands; the file-on-disk path is robust. `/tmp/` persists across runs on the same machine.