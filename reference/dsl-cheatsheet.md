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

## Editing

| Operation | When to use | Example |
|-----------|-------------|---------|
| `draft.edit(old, new)` | Know *what* to change | `draft.edit("old text", "new text")` |
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
| `compare(base)` | Compare branches | `my-project.compare(base=main)` |
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