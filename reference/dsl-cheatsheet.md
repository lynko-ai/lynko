---
title: Lynko DSL Cheatsheet
audience: users
last_validated: 2026-09-04
---

# Lynko DSL Cheatsheet

Quick reference for all Lynko operations.

## Syntax

```
collection.operation()                       # Collection-level operation
`collection`[path/to/file].operation()       # File-level operation
`collection`[directory/].operation()         # Directory-scoped operation
`collection`[**/*.go].operation()            # Glob-scoped operation
```

**Naming rules:**
- Simple names work directly: `my_project.ls()`
- Names with hyphens work directly: `my-project.ls()`
- Names with dots need backticks: `` `my-file.md`.read() ``
- Paths use bracket notation: `repo[src/main.go].read()` (backticks optional for simple names)

**Arguments:**
- Quotes optional for simple values: `ls(src/)` or `ls("src/")`
- Quotes required for spaces, commas, equals: `grep("hello world")`
- Named args: `grep("pattern", context_lines=3)`
- Raw strings for multi-line: delimit with `@@@@@` on own lines

## Path Resolution

**Bracket notation** resolves in this order:

1. Contains `*` → **glob scope** (`my-project[**/*.go].grep("TODO")`)
2. Ends with `/` → **directory scope** (`my-project[src/].grep("TODO")`)
3. Exact file match → **file reference** (`my-project[src/main.go].read()`)
4. Prefix matches a directory → **directory scope** (`my-project[src].grep("TODO")` — trailing slash optional)
5. None of the above → **partial match** across all files

**Partial path matching** (Single Match Resolution):

```
my-project[handler.go].read()               # Resolves to src/internal/handler.go
my-project[0001].read()                      # Resolves if only one file contains "0001"
my-drive["PromissoryNote"].read()            # Resolves to full Drive path
```

Matching is case-insensitive. If multiple files match, you'll get suggestions. Section names resolve the same way: `section("setup")` matches "Setup Guide" if it's the only match.

**Auto-resolve across collections:**

```
`src/main.go`.outline()                      # Resolves to the right collection automatically
```

If a path uniquely identifies a file across all collections in the pod, you don't need to specify the collection name. If ambiguous, you'll get an error showing which collections matched.

## Navigation

| Operation | What it does | Example |
|-----------|-------------|---------|
| `ls(path?)` | List files | `my-project.ls("src/")` |
| `tree(path?)` | Directory structure | `my-project.tree()` |
| `find(pattern)` | Find files by glob | `my-project.find(*.go)` |
| `toc()` | Table of contents (markdown) | `my-project[README.md].toc()` |
| `outline()` | Code structure (types, functions) | `my-project[main.go].outline()` |
| `sheets()` | Sheet order with each sheet's used range | `my-project[model.xlsx].sheets()` |

**Content types use different navigation patterns:**

| Content | Structure | Targeted | Full |
|---------|-----------|----------|------|
| Code | `outline()` | `expand("Symbol")`, `lines("10-20")` | `read()` |
| Markdown/Docs | `toc()` | `section("Heading")`, `lines("10-20")` | `read()` |
| PDF | `toc()` | `pages("1,3-5")` | `read()` |
| Google Docs | `toc()` | `section("Heading")`, `paragraphs("1-5")`, `tab("Tab 1")` | `read()` |
| Spreadsheets | `sheets()` | `rows("Sales!1-10")`, `cells("Sales!B5:D15")` | `read()` |
| Collections | `ls()`, `tree()` | `grep()`, `find()` | — |

## Reading

> All read operations show your current draft state when one exists, otherwise committed content. There is no separate "read the draft" namespace — `read()`, `grep()`, `section()`, etc. always reflect the working copy. Use `diff()` to compare draft vs committed.

| Operation | What it does | Example |
|-----------|-------------|---------|
| `read()` | Full file content | `my-project[README.md].read()` |
| `section(name)` | One section of a document | `my-project[guide.md].section("Setup")` |
| `expand(symbol)` | One function/type body | `my-project[main.go].expand("Handler")` |
| `lines(spec)` | Specific line range | `my-project[main.go].lines("10-30")` |
| `pages(spec)` | PDF page range | `my-project[doc.pdf].pages("1-3")` |
| `paragraphs(spec)` | Google Doc paragraph range | `my-drive[Roadmap].paragraphs("1-5")` |
| `tab(name)` | Google Doc tab by title (multi-tab only) | `my-drive[Roadmap].tab("Q4 Plan")` |
| `sheets()` | Sheet order with each sheet's used range | `my-project[model.xlsx].sheets()` |
| `rows(spec, columns=)` | Physical row range, optional column projection | `my-project[model.xlsx].rows("Sales!1-10")` |
| `cells(spec)` | Cell rectangle via A1 notation | `my-project[model.xlsx].cells("Sales!B5:D15")` |

**Hierarchical section paths:** when sections share common titles (like "Introduction" in multiple chapters), use ` > ` to disambiguate:

```
my-project[paper.md].section("Neural Networks > Estimation Error")
my-project[docs/api.md].section("API Design > Error Handling")
```

Intermediate levels can be skipped — `section("Neural Networks > Estimation Error")` resolves even without naming every parent.

**Reading PDFs:** PDFs use pages as their natural unit. `pages()` defaults to extracted text and falls back to rendered PNG for image-only pages. Override with `as="image"` for text pages whose content is tabular or visually complex — text extraction can misread multi-column layouts.

| Call | Use when |
|------|----------|
| `pages("1-5")` | Prose, narrative notes, MD&A; locating content |
| `pages("2", as="image")` | Multi-column tables, financial statements, charts; text shows glyph drops |
| `pages("2", as="text")` | Forcing text on an image-only page (errors if unavailable) |

`toc()` and `section()` work on PDFs with bookmarks. `grep()` searches extracted text from text pages; for image-only PDFs, `grep()` is hidden and `pages()` renders page images. Image mode runs ~2–4× the input-token cost of text at 150 DPI — use as a precision step, not the default.

**Reading Google Docs:** Google Docs in Drive collections use paragraphs as their natural unit, with headings forming a hierarchical table of contents. Multi-tab documents expose each tab as a navigable unit.

```
my-drive[Roadmap].toc()                      # Heading hierarchy (+ tab list for multi-tab)
my-drive[Roadmap].section("Phase 1")         # One section by heading
my-drive[Roadmap].paragraphs("1-5")          # Paragraph range (single-tab)
my-drive[Roadmap].paragraphs("Tab 1:1-5")    # Tab-qualified (multi-tab)
my-drive[Roadmap].tab("Q4 Plan")             # Full content of one tab
my-drive[Roadmap].grep("milestone")          # Search with section + tab context
```

`tab()` is advertised only on multi-tab documents. On single-tab Google Docs, `read()` returns the full document — the tab operation is hidden because it would be a no-op.

**Reading spreadsheets:** spreadsheets use physical rows and columns as their natural unit. An `.xlsx` file and a Google Sheet answer the same operations — `sheets()` shows the structure, `rows()` navigates data, `cells()` is the A1 escape hatch.

```
my-project[model.xlsx].sheets()                           # Sheet order and used ranges
my-project[prices.xlsx].rows("1-10")                      # First 10 rows (single-sheet workbook)
my-project[model.xlsx].rows("Sales!20-40", columns="F:H") # Sheet-qualified + column projection
my-project[model.xlsx].cells("Sales!B5:D15")              # A1-notation access
my-drive[Budget].rows("Sales!20-40")                      # The same calls on a Google Sheet
```

Sheet qualification is `<sheet>!<coordinate>`, for `rows()` and `cells()` alike. Multi-sheet workbooks require it — Lynko never silently picks the first sheet — and single-sheet workbooks may omit it. A title carrying spaces or delimiters accepts spreadsheet-style single quotes, and a title containing `!` requires them: `rows("'Revenue Model'!20-40")`.

Addresses are physical coordinates. Row 1 is the first worksheet row even when it holds headers, and `columns=` accepts column letters only — `"F:H"` or `"A,C,E:G"`. Header names are not selectors.

**Representation planes.** Reads show displayed values by default. Where a provider stores more than the displayed text, `as=` selects another plane, on `read()`, `rows()` and `cells()` alike:

| Call | What it shows |
|------|---------------|
| `cells("Sales!B5:D15")` | displayed values |
| `cells("Model!B20:D21", as="formulas")` | formula expressions, with literal cells shown as their values |
| `cells("Model!B20:D21", as="raw")` | stored typed scalars, unformatted |

`.xlsx` workbooks serve both extra planes. A Google Sheet holds display text only, so it advertises neither and refuses a direct request rather than passing display text off as a stored value. Lynko does not evaluate formulas: a workbook whose stored calculation state asks to be recalculated says so on the response's scope or header line.

`.xlsx` is the native workbook format. `.xls`, `.xlsm` and `.xlsb` are not read as spreadsheets.

## Searching

| Operation | What it does | Example |
|-----------|-------------|---------|
| `grep(pattern)` | Search file content | `my-project.grep("TODO")` |
| `grep(pattern, context_lines=N)` | Search with context | `my-project.grep("error", context_lines=3)` |
| `grep(/regex/)` | Regex search | `my-project.grep(/error\|warn/)` |
| `search(query)` | Semantic search — ranked by meaning, not exact text | `my-project.search("how auth tokens are refreshed")` |
| `find_definition(symbol)` | Find where defined | `my-project.find_definition("UserService")` |
| `find_references(symbol)` | Find all usages | `my-project.find_references("UserService")` |

**Searching spreadsheets:** `grep()` on a workbook matches cells, and every result is a canonical cell coordinate that chains straight into `cells()`. Collection-level `grep()` searches workbooks alongside text files, in git and Drive collections alike.

```
my-project[model.xlsx].grep("Widget")             # Sales!D37   Widget
my-project[model.xlsx].grep("P65", as="formulas") # Search the formula representation
my-project[model.xlsx].grep("45306", as="raw")    # Search stored typed values
my-project.grep("Widget")                         # Collection sweep — workbooks answer cells
```

`grep()` searches the representation `read()` renders, named the same way: whatever is visible at a cell under `read(as=R)` is findable with `grep(text, as=R)`. Under `as="formulas"` that includes literal cells — they render their own values there, so a workbook's formula representation is the whole grid, not just its expressions.

`as="formulas"` and `as="raw"` are offered only where the provider stores those representations. Where a cell cannot be stated in the one you asked for — a shared-formula follower carries no expression of its own — it is excluded from the search and counted beside the total, so a workbook never answers a bare "no matches" when something went unsearched. Spreadsheet `grep()` takes `max_results` but not `context_lines`: the coordinate is the whole result, and a neighbourhood is read with `rows()` or `cells()`. A collection sweep searches displayed values; `as=` is a file-level selector.

**Scope narrowing:**

```
my-project[src/].grep("pattern")             # Under a directory
my-project[**/*.go].grep("pattern")          # Only .go files (any depth)
my-project[*.md].grep("pattern")             # Only .md files (root level)
my-project[docs/**/*.md].search("auth flow") # Semantic search, scoped
```

**Which operations support scoping:**

| Operation | Directory scope | Glob scope |
|-----------|:-:|:-:|
| `grep` | ✓ | ✓ |
| `find` | ✓ | ✓ |
| `search` | ✓ | ✓ |
| `ls` | ✓ | — (use `find`) |
| `find_definition` | ✓ | — (use dir scope) |
| `find_references` | ✓ | — (use dir scope) |

Scope provides defaults — explicit arguments override. For example, `my-project[docs/].ls("adr")` lists `adr/`, not `docs/adr/`.

`search()` is on by default and builds its index on first use — a fresh collection may answer "building" once; retry in a moment. Tune with `max_results` and `vector_min_score` (a 0–1 similarity floor).

## Editing

| Operation | When to use | Example |
|-----------|-------------|---------|
| `draft.edit(old, new)` | Know *what* to change | `draft.edit("old text", "new text")` |
| `draft.edit(old, new, all=true)` | Replace *all* occurrences | `draft.edit("TODO", "DONE", all=true)` |
| `lines(N).draft.replace(new)` | Know *where* to change | `lines("42").draft.replace("new content")` |
| `draft.append(content)` | Add to end of file | `draft.append("\n\n## New Section")` |
| `draft(content)` | Create new file | `my-project[new.md].draft("# Title")` |
| `mv(destination)` | Rename/move file | `my-project[old.go].mv("new.go")` |
| `rm()` | Delete file | `my-project[unused.go].rm()` |

**Scoped edits** (match within a specific scope):

```
my-project[file.go].expand("Handler").draft.edit("old", "new")
my-project[guide.md].section("Setup").draft.edit("old", "new")
my-project[file.go].lines("10-20").draft.edit("old", "new")
```

**Which operations support edit scoping:**

- ✅ `section()`, `expand()`, `lines()` — continuous source regions
- ❌ `toc()`, `outline()`, `grep()` — derived views, not editable scopes

`replace()` is lines-only — `section().draft.replace()` and `expand().draft.replace()` are not supported.

**No recursive scoping:** `section(...).lines(...).draft.edit(...)` is not supported. Use one scope level per edit.

**Batched edits:** consecutive `draft.edit()` or `draft.replace()` calls on the same file in one multi-command call auto-batch with snapshot semantics — all line numbers resolve against the original content.

**Raw strings** (no escaping needed for multi-line):

```
my-project[file.go].draft.edit(
@@@@@
func old() {
    return nil
}
@@@@@
,
@@@@@
func new() {
    return result
}
@@@@@
)
```

Opening and closing `@@@@@` must each be on their own line. The comma between arguments also goes on its own line.

**Delimiter collision:** If your content itself contains `@@@@@`, use an alternative delimiter. Available delimiters are `@@@@@`, `@@@`, `"""`, and `'''`. Pick whichever one does not appear in your content. Example — editing a file that documents the `@@@@@` syntax:

```
my-project[docs/dsl-guide.md].draft.edit(
"""
old content that mentions @@@@@ delimiters
"""
,
"""
new content that mentions @@@@@ delimiters
"""
)
```

## Review & Version Control

| Operation | What it does | Example |
|-----------|-------------|---------|
| `status()` | Show pending drafts | `my-project.status()` |
| `diff()` | Show all changes | `my-project.diff()` |
| `commit(message)` | Commit and push | `my-project.commit("feat: add auth")` |
| `push()` | Rescue only: complete a commit whose push half failed | `my-project.push()` |
| `log(max?)` | Commit history | `my-project.log(max=20)` |
| `log(commit=hash)` | One commit's full message | `my-project.log(commit="abc1234")` |
| `compare(base)` | Stat summary vs branch | `my-project.compare(base=main)` |
| `compare(base, mode=patch)` | Full patch vs branch | `my-project.compare(base=main, mode=patch)` |
| `pull()` | Pull latest changes | `my-project.pull()` |
| `merge(branch)` | Merge branch into current | `my-project.merge("main")` |
| `restore()` | Discard all drafts | `my-project.restore()` |
| `draft.discard()` | Discard one file's draft | `my-project[file.go].draft.discard()` |

**`push()` is a rescue, not a step.** `commit()` already pushes. If a `commit()` call errors but `log()` shows the commit, the commit landed and its push half failed (a remote-side failure): reads then serve pre-commit content for the affected paths, a later `commit()` touching them is refused with "path(s) changed on the branch since your draft was authored", and `pull()` is a no-op because origin is behind, not ahead. Confirm with `log()` and `compare(base=<origin tip>)`, then run `push()`.

## Testing

`test()` runs against your **draft content** — no commit needed. Test before you commit, not after.

```
my-project.test(targets="my-component")      # Run tests on current drafts
```

`test()` returns immediately with a run ID. Navigate results top-down:

```
ci.history()                                 # List recent runs with status
ci.status()                                  # Testable collections + active runs
ci["run-ID"].ls()                            # List results (passed/failed/skipped)
ci["run-ID/34"].read()                       # Read specific result by index
ci["run-ID"].read(result_index="test-unit: my-component")  # Read by name
ci["run-ID"].grep("FAIL", context_lines=3)   # Search across all results
```

## Running Commands

`run()` dispatches a command to a target machine via an attached runner node. Requires a runner node and at least one machine configured on your pod. Unlike `test()`, it runs against the current git state and doesn't need to be scoped to a CI build.

```
my-project.run("cd .. && ls")                # Dispatch a command; returns a run ID
my-project.run("pytest -k widget")           # Any shell command
my-project[src/].run("go build ./...")       # Scope to a subdirectory
my-project.run("./deploy.sh", machine="prod", timeout="10m")   # Pick a machine, set timeout (Go duration: "30s", "5m", "1h30m")
```

`run()` returns immediately with a run ID. Poll results with runner ops:

```
runner.status()                              # Configured machines + active runs
runner.history()                             # Recent runs + state and duration (newest first)
runner["run-ID"].read()                      # Full captured output
runner["run-ID"].lines("1-50")               # Head of output (range)
runner["run-ID"].lines("100-120")            # Specific line range
runner["run-ID"].grep("ERROR", context_lines=3)   # Search captured output
runner["run-ID"].cancel()                    # Cancel a running command
```

During execution, `read()` / `grep()` / `lines()` query the target machine live over SSH. Once the run completes, output is captured to your pod and these operations read from the database — they keep working even if the machine is offline.

### Tips

- Combine with the full DSL: `my-project[src/server.go].draft.edit(...)` → `my-project.test(...)` → `my-project.run("./smoke.sh")` all run against your current drafts on writable collections.
- Two execution paths: with drafts → checkpoint to a system branch, push, target fetches checkpoint; without drafts → fast path, target fetches the tracked branch directly. Drafts on a read-only collection (no push access) are rejected with `RUNNER_ERROR_CANNOT_CHECKPOINT`.
- Long-running commands survive client disconnects — the run continues on the machine (via tmux). Use `runner.status()` while it is active and `runner.history()` for its final state and duration.

```
artifacts()                                  # List all collections and operations
```

## Skills

Requires a skills node attached to the pod. Every qualified `SKILL.md` (frontmatter with `name` + `description`) across your collections becomes one searchable set:

```
skills.ls()                                  # All skills, grouped by collection
skills["my-repo/"].ls()                      # Skills from one collection
skills.search("deploy a staging build")      # Semantic search across all skills
skills.search("smoke test", pinned=true)     # Only pinned skills
skills["my-repo/ops/deploy/SKILL.md"].info() # One skill's details
skills["my-repo/ops/deploy/SKILL.md"].read() # Full SKILL.md content
skills.status()                              # Index health + excluded count
skills.status(state="excluded")              # List skills hidden by curation
```

Curation — per-pod rules that shape discovery without touching the underlying repos:

```
skills["my-repo/ops/deploy/SKILL.md"].pin()      # Boost in search results
skills["my-repo/ops/deploy/SKILL.md"].unpin()    # Remove the boost
skills["**/experiments/**"].exclude()            # Hide from ls/search (globs ok)
skills["my-repo/experiments/keeper/SKILL.md"].include()  # Re-surface under a broader exclude
skills["**/experiments/**"].reset()              # Clear rules at a selector (both axes)
skills.curation()                                # List all rules on this pod
```

### Tips

- Pods start with hygiene defaults that exclude `**/test/fixtures/**` and `**/testdata/**` — fixture skills won't appear in `ls()`/`search()`. Use `skills.curation()` to see the rules and `include()` to re-surface specific paths.
- `exclude` hides skills from discovery only — an excluded skill still reads fine by exact path (`info()`/`read()`).
- The most specific selector wins; an exact-path `include` beats a broader `exclude`. Write responses report how many skills actually flipped visibility.
- Rules are per-pod: curating here never affects other pods sharing the same collections.

## Connection Roles

A role is a named, pod-local permission set — read/write/admin grants over collections and paths, with write modes `none` / `append` / `full`. Every pod has the implicit full-access `author` role; define scoped roles (e.g. `reviewer`) per pod in the dashboard under **Pods → Roles**.

Point an agent at a role by giving it the role's connection URL instead of the plain pod URL:

```
https://mcp.lynko.ai/pods/<pod-id>/as/<role>
```

The connection then carries only that role's access — operations outside the role don't appear in `artifacts()`, and attempts are rejected server-side. Check any connection with:

```
whoami()                                     # Role, read/write/admin, max write mode, scope summary
```

Roles are also what `invoke()` dispatches sub-agents as — see [Invoking Agents](#invoking-agents).

## Invoking Agents

`invoke()` dispatches a **sub-agent** into the collection's pod — the agent-to-agent sibling of `run()` and `test()`. The chosen agent driver runs your prompt under a role you pick, scoped to be **no more powerful than your own** connection. Canonical use: have a sub-agent review your own diff.

```
my-project.invoke("claude-code", "reviewer", "Review the diff on this branch and flag risky changes")
my-project.invoke("codex", "reviewer", "Summarize what changed and call out anything Tier 3", timeout="10m")
```

| Argument | What it is |
|----------|------------|
| `agent` | The driver to launch: `codex` or `claude-code`. |
| `as` | A **concrete role** the sub-agent runs as (e.g. `reviewer`) — see [Connection Roles](#connection-roles). Its authority is attenuated to a subset of yours — never the full / `author` role. |
| `prompt` | The instruction handed to the sub-agent. |
| `timeout` | Optional. Go duration (`"30s"`, `"10m"`, `"1h"`); the sub-agent is run-bounded and capped at 1h. |

Positionals bind in order (`agent`, `as`, `prompt`, `timeout`); named args must come after any positionals — `invoke("codex", "reviewer", "…", timeout="30m")` or fully named `invoke(agent="codex", as="reviewer", prompt="…")`. A named arg before a positional mis-binds. For multi-line prompts, pass a `@@@@@` raw string.

`invoke()` returns immediately with a run ID — the invocation is a normal runner run. Observe it through the same `runner[...]` surface as `run()`:

```
runner.status()                              # Active runs + installed_agents (which drivers are available)
runner.history()                             # Recent runs + final state and duration
runner["run-ID"].read()                      # Full sub-agent output
runner["run-ID"].lines("1-50")               # Head of output (range)
runner["run-ID"].grep("FAIL", context_lines=3)   # Search the sub-agent's output
runner["run-ID"].cancel()                    # Terminate the sub-agent
```

The sub-agent runs **attenuated** to `as`, **non-chaining** (it cannot itself `invoke()` another agent), and **run-bounded** (it terminates with the run). It already knows where it is: an invocation context block naming its collection and role is prepended to your prompt, so spend the prompt on the task, not orientation. Check `runner.status()` → `installed_agents` to see which drivers the pod has available before invoking.

## Glob Patterns

| Pattern | Matches | Example |
|---------|---------|---------|
| `*` | Any chars within one segment | `*.go` matches `main.go` (not `src/main.go`) |
| `**` | Zero or more path segments | `**/*.go` matches `main.go`, `src/main.go` |
| `docs/*` | Direct children of docs/ | `docs/api.md` only |
| `docs/**` | All descendants of docs/ | `docs/api.md`, `docs/v1/api.md` |

## Multi-Command

Put multiple commands on separate lines in one call:

```
my-project[src/server.go].outline()
my-project[src/handler.go].outline()
my-project[src/].grep("TODO")
```

All execute in a single round trip.