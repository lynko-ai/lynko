---
title: FAQ
audience: users
last_validated: 2026-07-14
---

# FAQ

## General

**What is Lynko?**
Lynko is a platform that exposes your content sources (git repos, Google Drive, PDFs) to AI agents through a unified DSL. Instead of downloading files or managing format-specific tools, agents navigate your content through operations like `read()`, `grep()`, and `outline()`.

**What AI clients work with Lynko?**
Any client that supports MCP (Model Context Protocol) custom connectors. This includes Claude (Desktop and web), ChatGPT, and many developer tools. See the [getting started guide](../getting-started.md) for setup instructions.

**What data sources can I connect?**
Git repositories (public and private) and Google Drive folders. PDF documents, markdown files, code files, and Google Sheets within those sources are all navigable. Sheets get dedicated operations — `sheets()`, `rows()`, `cells()` — for structured row-based access.

**Is my data safe?**
Lynko syncs content from your sources (GitHub, Google Drive) so agents can navigate it. Your original data remains under your control in its source platform. All access is authenticated through your account and scoped to your workspace.

**How do I give an agent limited access — say, read-only?**
Define a scoped role on the pod (dashboard → your pod's **Roles** button): a named set of read/write/admin grants over collections and paths. Then point that agent at the role's connection URL — `https://mcp.lynko.ai/pods/<pod-id>/as/<role>` — instead of the plain pod URL. The connection carries only what the role grants: operations outside it don't appear in `artifacts()`, and attempts are rejected server-side. Verify with `whoami()`. The same roles are what `invoke()` dispatches sub-agents as.

**My agent already has bash, grep, and file access. What does Lynko add?**
Bash + grep gets you bytes. Lynko gets you *structural navigation* — the same way humans read: tables of contents, section headings, function outlines. An agent forced to rebuild these per-session (parsing PDFs with custom scripts, extracting markdown TOCs by hand, running ad-hoc AST tools) is essentially rebuilding Lynko — slowly, incompletely, and from scratch every time. Lynko ships the navigation once; every agent can use it immediately, across every content type and every source.

**My agent already has a Google Drive or GitHub connector. What does Lynko add?**
Native connectors move bytes; Lynko exposes handles. A Drive or GitHub connector returns a snippet or file; Lynko returns a live section, function, or row address the agent can revisit and cite across steps. On writable Git collections, supported structural handles also bound edits, and the same surface carries drafts through test and commit. On read-only sources, handles remain precise navigation targets.

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
`run()` requires a runner node attached to your pod with at least one machine configured. If you only see CI-powered collections with `test()`, the runner node isn't attached yet. The runner is opt-in — bring your own SSH target.

**Do I need to commit before running `run()`?**
No. On writable collections, `run()` automatically checkpoints your drafts to a system branch before dispatching, so the machine sees your in-progress changes. When there are no drafts, `run()` uses a fast path and runs against the latest committed state — this works on both writable and public/read-only collections. Drafts on a read-only collection are rejected with `RUNNER_ERROR_CANNOT_CHECKPOINT` since there's no remote to push the checkpoint to.

**What's `invoke()` and how is it different from `run()`?**
`run()` executes a shell command; `invoke()` launches a whole sub-agent (`codex` or `claude-code`) inside the pod under a role you pick — always a subset of your own authority, never the full `author` role. The sub-agent can't `invoke()` further (non-chaining) and terminates with its run (capped at 1h). Dispatch with `my-project.invoke("codex", "reviewer", "…")`, then read its output like any run: `runner["run-ID"].read()`. Requires a runner node with the agent installed — check `runner.status()` → `installed_agents`.

**My `run()` output is long — how do I read it efficiently?**
Use `runner["run-id"].status()` for the summary, `runner["run-id"].lines("N-M")` for specific ranges, and `runner["run-id"].grep("pattern")` to search. Reserve `read()` for short output — `read()` returns the full captured log and has no range parameter. Very long outputs are bounded at ~10MB (head+tail split past the limit) — if you need the middle, scope the command to produce less output or redirect to a file on the target and read it with a follow-up run.

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
Verify the URL is exactly `https://mcp.lynko.ai/` (with trailing slash).

**Operation returns "not found"**
Check the file path. Use `ls()` to see available files, or `find(*pattern*)` to search. Paths are case-sensitive.

**Edits seem lost between sessions**
Drafts persist across sessions within the same pod. Run `status()` at the start of each session to check for pending drafts. If you switched pods, drafts are in the other pod.

**"Operation not available for this artifact"**
Some operations are only advertised when the artifact's content supports them — e.g., `toc()` and `section()` on a PDF require bookmarks, `grep()` on a PDF requires extractable text, and `tab()` on a Google Doc requires multiple tabs. Run `my-collection[path/to/file].ls()` to see which operations are available for that specific artifact (shown as `Try: ...` in the AX response). For image-only PDFs, use `pages()`; for single-tab Google Docs, `read()` returns the full document.