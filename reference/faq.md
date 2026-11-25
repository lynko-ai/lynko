---
title: FAQ
audience: users
last_validated: 2026-11-25
---

# FAQ

## General

**What is Lynko?**
Lynko gives agents a shared environment over your existing organizational content and attached capabilities. A pod groups the sources, roles, and working state for a piece of work; agents interact with it through a structural DSL over MCP.

**What AI clients work with Lynko?**
Any client that supports MCP (Model Context Protocol) custom connectors. This includes Claude (Desktop and web), ChatGPT, and many developer tools. See the [getting started guide](../getting-started.md) for setup instructions.

**What data sources can I connect?**
Git repositories (public and private) and Google Drive folders. PDF documents, markdown files, code files, and Google Sheets within those sources are all navigable. Sheets get dedicated operations — `sheets()`, `rows()`, `cells()` — for structured row-based access.

**Is my data safe?**
Lynko syncs content from your sources (GitHub, Google Drive) so agents can navigate it. Your original data remains under your control in its source platform. All access is authenticated through your account and scoped to your workspace.

**How do I give an agent limited access — say, read-only?**
Define a scoped role on the pod (dashboard → your pod's **Roles** button): a named set of read/write/admin grants over collections and paths. Then point that agent at the role's connection URL — `https://mcp.lynko.ai/pods/<pod-id>/as/<role>` — instead of the plain pod URL. The connection carries only what the role grants: operations outside it don't appear in `artifacts()`, and attempts are rejected server-side. Verify with `whoami()`. The same roles are what `invoke()` dispatches sub-agents as.

**My agent already has bash, grep, and file access. What does Lynko add?**
A local filesystem is excellent for one colocated agent. Lynko adds the parts that matter when the work spans remote heterogeneous sources or multiple agent surfaces: stable source identity, structural handles, server-enforced roles, persistent pod drafts/runs/results, and shared access to the same environment across sessions and harnesses. Operations such as `toc()`, `section()`, `outline()`, and `rows()` also keep agents from rebuilding content-specific parsers and ad-hoc context themselves.

**My agent already has a Google Drive or GitHub connector. What does Lynko add?**
A native connector gives an agent access to one source. Lynko makes connected sources part of one shared working environment: stable structural handles, scoped roles, persistent draft/run state, and attached capabilities such as CI and Runner. Native source systems remain authoritative.

**Why MCP, not a typed API like gRPC?**
MCP is a text-in, text-out protocol. That matches how agents actually reason — they were trained on text and think in text. A rigid typed API would force the agent to translate between its natural mental model and a schema, wasting attention on protocol plumbing. MCP lets the agent stay in one mental mode. It's also the emerging standard — Claude, ChatGPT, Cursor, and most major AI clients support custom MCP connectors. Connect once, work everywhere.

## DSL

**What's the difference between `read()` and `section()`?**
`read()` loads the entire file. `section("name")` loads just one section of a markdown document. For large files, prefer `toc()` first to see the structure, then `section()` for what you need. This keeps context usage minimal.

**What's the difference between `draft.edit()` and `lines().draft.replace()`?**
`draft.edit("old", "new")` finds exact text and replaces it — content-addressed. `lines("42").draft.replace("new")` replaces by line number — position-addressed. Use `draft.edit()` as primary; fall back to `lines().draft.replace()` when exact text matching is tricky.

**Why did `draft.edit()` fail?**
The old text must match exactly, including whitespace. Use `grep()` or `lines()` to find the exact current text, then retry. Or use `lines("N").draft.replace()` which doesn't require content matching.

**What does `@@@@@` mean?**
It's a raw string delimiter. Content between `@@@@@` markers is passed through literally with no escaping needed. Use it for multi-line code edits. The opening and closing `@@@@@` must each be on their own line.

**Do read operations see my drafts, or just committed content?**
All read operations — `read()`, `grep()`, `lines()`, `section()`, `expand()`, `outline()`, `toc()`, `pages()`, etc. — transparently see your draft state when one exists, otherwise committed content. There's no separate "read the draft" namespace; reads always show the current working state. Only writes use the `.draft.` prefix (`draft.edit`, `draft.append`, `draft.replace`). After a sequence of edits, you can verify the result with the same operation you'd normally use: `[file].grep("pattern")` reflects your draft. Use `diff()` to see what's changed vs committed; use `status()` to list files with pending drafts.

**Do I need to commit before running tests?**
No. `test()` runs against your current draft content. Test first, commit after tests pass.

**Do I need to push after committing?**
No. `commit()` handles both commit and push in one operation.

**How does `merge()` work?**
`merge("main")` merges `main` into your current branch. Auto-resolved files and conflicts all become drafts. Conflicts contain `<<<<<<` / `>>>>>>` markers that you resolve with `draft.edit()`. After resolving, `commit()` to finalize. It's not a GitHub PR — it merges directly and lets you resolve conflicts in-place.

**Can multiple agents edit the same collection?**
Drafts are scoped to your pod. Multiple agents using the same pod share draft state. For isolated work, use separate pods.

**What's the difference between `test()` and `run()`?**
`test()` runs your project's CI suite — targets come from your CI config (Makefile, GitHub Actions, etc.). `run()` executes any shell command on a target machine attached to the pod. Use `test()` for the canonical test set, `run()` for builds, scripts, inspections, or smoke tests. Both run against your current drafts on writable collections.

**Why doesn't `run()` show up on my collection?**
`run()` is exposed when Runner is attached to the pod; use `artifacts()` to discover its actual reference name. Attach Runner by reference name in the dashboard. Dispatch also needs at least one SSH machine of the pod's owner, added in the dashboard's [Compute panel](dashboard.md#compute-panel) and ticked for this pod on its [Runner page](dashboard.md#pod-runner-page); with none ticked, dispatch is refused rather than the operation hidden. A machine added later reaches no pod until it is ticked, and a machine that is not ticked is refused exactly like a name that does not exist. With several, select `machine="name"` and inspect it with `status(machine="name")`. Each machine needs `git` and `tmux` 3.3a or newer on its SSH account's `PATH`; see [Machine requirements](dashboard.md#machine-requirements).

**Do I need to commit before running `run()`?**
No. `run()` executes its exact recorded source: the tracked commit plus any current staged changes. Your drafts stay staged; the run does not commit or discard them, or publish the source to the remote. With no effective staged changes, it runs the recorded tracked commit on writable or read-only remote collections and does not follow later advances of the remote branch. Staged source can also run when a previously pushable remote now refuses pushes, while the collection remains recorded as supporting pushes. Creating new drafts on a collection without recorded push support remains unavailable.

**What's `invoke()` and how is it different from `run()`?**
`run()` executes a shell command; `invoke()` launches `codex`, `claude` (also accepted as `claude-code`) or `opencode` under a concrete same-pod role that is a subset of your authority, never `author`/full. Choose a machine ticked for the pod with `machine="name"` when several are, and inspect its installed agents first. The child cannot invoke again or refresh its delegated credential. The requested timeout defaults to 3600 seconds; credential life minus 30 seconds and configured Runner limits can shorten the scheduled window. Like `run()`, an invocation takes one of the pod's slots on the machine, waits its turn when they are busy, and counts toward the 16-run limit. Follow the returned run ID through Runner status/history and bounded output. See [Invoking Agents](dsl-cheatsheet.md#invoking-agents) for the seven named arguments, four positional slots, profiles and exact timeout grammar; requested settings are not observed provider evidence.

**My `run()` output is long — how do I read it efficiently?**
Use `runner.status()` to see active runs and `runner.history()` to see recent runs with their state and duration. For a specific run, use `runner["run-id"].lines("N-M")` for ranges and `runner["run-id"].grep("pattern")` to search. Reserve `read()` for short output — `read()` returns the full captured log and has no range parameter. Very long outputs are bounded at ~10MB (head+tail split past the limit) — if you need the middle, scope the command to produce less output or redirect to a file on the target and read it with a follow-up run.

**Why don't some skills show up in `skills.ls()` or `skills.search()`?**
Pods come with hygiene defaults that exclude `**/test/fixtures/**` and `**/testdata/**` from discovery, and you (or an agent) may have added curation rules. Run `skills.curation()` to see every rule on the pod. Re-surface a path with `skills["path"].include()` — an exact-path include wins over a broader exclude — or clear a rule with `reset()`. Exclusion affects `ls()`/`search()` only: any skill still reads by exact path. `skills.status(state="excluded")` lists everything currently hidden.

**What makes a file a skill?**
A `SKILL.md` file with frontmatter containing a `name` and a `description`. Qualification is checked when the commit lands — the commit response tells you when a file newly qualifies — and the skill appears in `skills.search()` after the next refresh.

**Why doesn't the `skills` surface show up in `artifacts()`?**
The skills node isn't attached to this pod. It's opt-in — add it from the dashboard (Pods → Nodes → + Add Node), pick a reference name, and the discovery and curation operations appear.

## Troubleshooting

**"No artifacts found" after connecting**
Make sure you've added at least one collection in your dashboard and it has finished syncing. Run `artifacts()` to see what's available.

**MCP connection fails**
Verify the URL is exactly `https://mcp.lynko.ai/mcp`.

**Operation returns "not found"**
Check the file path. Use `ls()` to see available files, or `find(*pattern*)` to search. Paths are case-sensitive.

**Edits seem lost between sessions**
Drafts persist across sessions within the same pod. Run `status()` at the start of each session to check for pending drafts. If you switched pods, drafts are in the other pod.

**"Operation not available for this artifact"**
Some operations are only advertised when the artifact's content supports them — e.g., `toc()` and `section()` on a PDF require bookmarks, `grep()` on a PDF requires extractable text, and `tab()` on a Google Doc requires multiple tabs. Run `my-collection[path/to/file].ls()` to see which operations are available for that specific artifact (shown as `Try: ...` in the AX response). For image-only PDFs, use `pages()`; for single-tab Google Docs, `read()` returns the full document.