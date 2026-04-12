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

## DSL

**What's the difference between `read()` and `section()`?**
`read()` loads the entire file. `section("name")` loads just one section of a markdown document. For large files, prefer `toc()` first to see the structure, then `section()` for what you need. This keeps context usage minimal.

**What's the difference between `draft.edit()` and `lines().draft.replace()`?**
`draft.edit("old", "new")` finds exact text and replaces it — content-addressed. `lines("42").draft.replace("new")` replaces by line number — position-addressed. Use `draft.edit()` as primary; fall back to `lines().draft.replace()` when exact text matching is tricky.

**Why did `draft.edit()` fail?**
The old text must match exactly, including whitespace. Use `grep()` or `lines()` to find the exact current text, then retry. Or use `lines("N").draft.replace()` which doesn't require content matching.

**What does `@@@@@` mean?**
It's a raw string delimiter. Content between `@@@@@` markers is passed through literally with no escaping needed. Use it for multi-line code edits. The opening and closing `@@@@@` must each be on their own line.

**Do I need to commit before running tests?**
No. `test()` runs against your current draft content. Test first, commit after tests pass.

**Do I need to push after committing?**
No. `commit()` handles both commit and push in one operation.

**How does `merge()` work?**
`merge("main")` merges `main` into your current branch. Auto-resolved files and conflicts all become drafts. Conflicts contain `<<<<<<` / `>>>>>>` markers that you resolve with `draft.edit()`. After resolving, `commit()` to finalize. It's not a GitHub PR — it merges directly and lets you resolve conflicts in-place.

**Can multiple agents edit the same collection?**
Drafts are scoped to your pod. Multiple agents using the same pod share draft state. For isolated work, use separate pods.

## Troubleshooting

**"No artifacts found" after connecting**
Make sure you've added at least one collection in your dashboard and it has finished syncing. Run `artifacts()` to see what's available.

**MCP connection fails**
Verify the URL is exactly `https://mcp.lynko.ai/` (with trailing slash). For Claude, ensure the Client ID and Client Secret are set correctly under Advanced Settings.

**Operation returns "not found"**
Check the file path. Use `ls()` to see available files, or `find(*pattern*)` to search. Paths are case-sensitive.

**Edits seem lost between sessions**
Drafts persist across sessions within the same pod. Run `status()` at the start of each session to check for pending drafts. If you switched pods, drafts are in the other pod.