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

## Reading

| Operation | What it does | Example |
|-----------|-------------|---------|
| `read()` | Full file content | `my-project[README.md].read()` |
| `section(name)` | One section of a document | `my-project[guide.md].section("Setup")` |
| `expand(symbol)` | One function/type body | `my-project[main.go].expand("Handler")` |
| `lines(spec)` | Specific line range | `my-project[main.go].lines("10-30")` |
| `pages(spec)` | PDF page range | `my-project[doc.pdf].pages("1-3")` |

**Hierarchical section paths:** when sections share common titles (like "Introduction" in multiple chapters), use ` > ` to disambiguate:

```
my-project[paper.md].section("Neural Networks > Estimation Error")
my-project[docs/api.md].section("API Design > Error Handling")
```

Intermediate levels can be skipped — `section("Neural Networks > Estimation Error")` resolves even without naming every parent.

## Searching

| Operation | What it does | Example |
|-----------|-------------|---------|
| `grep(pattern)` | Search file content | `my-project.grep("TODO")` |
| `grep(pattern, context_lines=N)` | Search with context | `my-project.grep("error", context_lines=3)` |
| `grep(/regex/)` | Regex search | `my-project.grep(/error\|warn/)` |
| `find_definition(symbol)` | Find where defined | `my-project.find_definition("UserService")` |
| `find_references(symbol)` | Find all usages | `my-project.find_references("UserService")` |

**Scope narrowing:**

```
my-project[src/].grep("pattern")             # Under a directory
my-project[**/*.go].grep("pattern")          # Only .go files (any depth)
my-project[*.md].grep("pattern")             # Only .md files (root level)
```

**Which operations support scoping:**

| Operation | Directory scope | Glob scope |
|-----------|:-:|:-:|
| `grep` | ✓ | ✓ |
| `find` | ✓ | ✓ |
| `ls` | ✓ | — (use `find`) |
| `find_definition` | ✓ | — (use dir scope) |
| `find_references` | ✓ | — (use dir scope) |

Scope provides defaults — explicit arguments override. For example, `my-project[docs/].ls("adr")` lists `adr/`, not `docs/adr/`.

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
| `log(max?)` | Commit history | `my-project.log(max=20)` |
| `log(commit=hash)` | One commit's full message | `my-project.log(commit="abc1234")` |
| `compare(base)` | Stat summary vs branch | `my-project.compare(base=main)` |
| `compare(base, mode=patch)` | Full patch vs branch | `my-project.compare(base=main, mode=patch)` |
| `pull()` | Pull latest changes | `my-project.pull()` |
| `restore()` | Discard all drafts | `my-project.restore()` |
| `draft.discard()` | Discard one file's draft | `my-project[file.go].draft.discard()` |

## CI / Testing

```
my-project.test(targets="my-component")      # Run targeted tests
ci["run-ID"].ls()                            # Check test results
ci["run-ID"].grep("FAIL", context_lines=3)   # Find failures
```

## Pod-Level Operations

```
artifacts()                                  # List all collections and operations
```

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