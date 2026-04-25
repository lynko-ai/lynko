---
name: lynko-run
description: >
  Execute commands on a target machine through Lynko — builds, scripts, one-off
  inspections, smoke tests. Use when you need to run something on a specific
  host (a preconfigured dev VM, build server, or test target) and capture the
  output for review. Requires a runner node attached to the pod with at least
  one machine configured.
---

# Run Commands with Lynko

For full syntax details, see the [DSL cheatsheet](../../reference/dsl-cheatsheet.md#running-commands).

## When to use `run()` vs `test()`

- **`test()`** — runs your project's CI suite. Targets come from the CI config (Makefile, GitHub Actions, etc.). Use when you want the canonical test set.
- **`run()`** — runs any shell command on a target machine. Use for builds, scripts, inspections, smoke tests, or anything that isn't a CI target.

Both run against your current drafts on writable collections (see [Checkpoint Behavior](#checkpoint-behavior)).

## Discovery

Start by checking what's available:

```
runner.status()                              # Configured machines + active runs
runner.history()                             # Recent runs (newest first)
```

If `runner.status()` fails with "node not attached," the runner isn't on this pod yet. If it succeeds but lists zero machines, attach one from the dashboard.

**Assume nothing is preinstalled.** Machines are typically minimal Linux (Ubuntu) by default — `bash`, `wget`, `apt` (without sudo), `python3`, and standard userland are usually available, but `node`, `curl`, language toolchains, and other dev tools are not. Probe with `which <tool>` before relying on anything beyond standard userland. To install a CLI without sudo, prefer downloading a static binary into `$HOME` and running it directly (e.g., `wget` a release zip, `unzip`, run from `$HOME/.local/bin/`).

## Dispatch and Poll

`run()` returns immediately with a run ID. Poll until terminal state before reading results:

```
my-project.run("go build ./...")
# → Run dispatched: run-20260417T1145-abc1
#   Check: runner["run-20260417T1145-abc1"].status()

# Wait a bit (seconds for fast commands, minutes for builds)
runner["run-20260417T1145-abc1"].status()
# → status=succeeded, exit_code=0, duration=12s
```

Terminal states: `succeeded`, `failed`, `canceled`. Anything else means still running.

`run()` is dispatched from any writable collection (`my-project.run(...)`), but the run is queryable through the global `runner[]` accessor (`runner["run-ID"].status()`). These reference the same execution — collection scope is the dispatch context, `runner[]` is the read interface.

**LLM-client polling pattern.** Most LLM clients (Claude, ChatGPT, Cursor, etc.) timeout tool calls at ~25 seconds, which is much shorter than typical build/test runs. The standard pattern is: dispatch with `run()`, then issue a separate `bash sleep N` (10-60s depending on expected duration) before calling `status()`. Don't try to block inside a single tool call — the client will time out before the runner finishes.

## Reading Output

Navigate output top-down — prefer `grep` and `lines` over full `read()` for long output:

```
runner["run-ID"].status()                    # Summary: state, exit code, duration, line count
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

Pick a specific machine when multiple are configured:

```
my-project.run("./deploy.sh", machine="prod")
my-project.run("./bench.sh", machine="perf-box", timeout="30m")
```

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

# 3. Poll (sleep a few seconds between checks for fast commands)
runner["run-ID"].status()

# 4. When done — review output
runner["run-ID"].grep("FAIL|ERROR", context_lines=3)   # Look for problems
runner["run-ID"].lines("1-30")                         # Check the start

# 5. Commit if it worked
my-project.commit("feat(handler): ...")
```

## Checkpoint Behavior

`run()` uses two paths based on draft presence:

- **No drafts (fast path)** — the target machine fetches the tracked branch and runs the command against the latest committed state. Used for both read-only public repos and writable repos with no pending changes.
- **Drafts present on a writable collection** — `run()` checkpoints your drafts to a system branch (`lynko/runner/<pod>/<collection>`) on the remote before dispatching, so the machine sees your in-progress changes. No separate commit step needed.
- **Drafts present on a read-only / public-repo collection** — rejected with `RUNNER_ERROR_CANNOT_CHECKPOINT`. There's no remote to push the checkpoint to. Either commit elsewhere first, or move the drafts to a writable collection.

## Tips

- Concurrent runs: one command per machine at a time. Dispatching a second run while the first is active returns a `RunInProgress` error with the active run ID — check or cancel it first.
- Long-running commands survive client disconnects. The run continues on the machine (via tmux); `runner["run-ID"].status()` reports the final state when the command finishes.
- `runner.history()` paginates — the output includes the cursor for the next page.
- Captured output is bounded at ~10MB (head+tail split past the limit, with a `--- OUTPUT TRUNCATED ---` marker between the halves). `read()`, `grep()`, and `lines()` all operate on the captured output; if truncation loses something you need, scope the command to produce less output or redirect to a file on the target and read it with a follow-up run.
- For multi-line scripts with complex escapes (Python heredocs, embedded quotes, `\n` in strings), prefer writing the script to `/tmp/` first and executing by path, rather than passing the script inline. Pattern: `cat > /tmp/foo.py << 'EOF' ... EOF && python3 /tmp/foo.py`. Inline heredocs occasionally trip the output-capture path on short-lived commands; the file-on-disk path is robust. `/tmp/` persists across runs on the same machine.