---
name: lynko-edit
description: >
  Edit content through Lynko — scoped edits, line replacements, new files,
  multi-line changes. Use when modifying code, updating documents, or making
  any changes to files in a collection.
---

# Edit with Lynko

## Core Concept: Navigate, Then Edit

The same operation that reads content also defines the edit scope. Read a function, then edit within it. Read a section, then edit within it.

```
my-project[src/server.go].expand("Handler")                    # Read the function
my-project[src/server.go].expand("Handler").draft.edit("old", "new")  # Edit within it
```

This is safer than file-level edits — the match is constrained to the scope, avoiding accidental replacements elsewhere.

## Editing Methods

### Content-addressed: `draft.edit()` (primary)

Match exact text and replace it:

```
my-project[README.md].draft.edit("old text", "new text")
```

Replace all occurrences (not just the first):

```
my-project[README.md].draft.edit("old text", "new text", all=true)
```

### Position-addressed: `lines().draft.replace()` (fallback)

When exact text matching is tricky, replace by line number:

```
my-project[src/server.go].lines("42").draft.replace("new line content")
my-project[src/server.go].lines("10-15").draft.replace("replacement block")
```

`replace()` is lines-only — `section().draft.replace()` and `expand().draft.replace()` are not supported. Use `section().draft.edit()` or `expand().draft.edit()` instead.

### Append: `draft.append()`

```
my-project[docs/guide.md].draft.append("\n\n## New Section\n\nContent here.")
```

`draft.append()` does raw concatenation — always include `\n` for newlines.

### Create new file: `draft()`

```
my-project[docs/new-guide.md].draft("# Getting Started\n\nIntroduction goes here.")
```

The file exists as a draft until you commit.

## Scoped Editing

Scope an edit to a specific region of the file:

| Scope | What it targets | Example |
|-------|----------------|---------|
| `section("name")` | Document section | `my-project[guide.md].section("Setup").draft.edit(...)` |
| `expand("symbol")` | Function or type body | `my-project[main.go].expand("Handler").draft.edit(...)` |
| `lines("N-M")` | Line range | `my-project[main.go].lines("10-20").draft.edit(...)` |

Only `section()`, `expand()`, and `lines()` support edit scoping — these return continuous source regions. Recursive scoping (e.g., `section().lines().draft.edit()`) is not supported. Use `section("API Design > Error Handling")` with ` > ` paths to disambiguate nested sections. See the [cheatsheet](../reference/dsl-cheatsheet.md) for the full constraints.

## Multi-Line Edits with Raw Strings

Use `@@@@@` delimiters — no escaping needed:

```
my-project[src/server.go].draft.edit(
@@@@@
func old() {
    fmt.Println("hello")
}
@@@@@
,
@@@@@
func new() {
    fmt.Println("world")
}
@@@@@
)
```

Delimiters must be on their own lines. See the [cheatsheet](../reference/dsl-cheatsheet.md) for delimiter collision handling when your content contains `@@@@@`.

## Batched Edits

Consecutive edits or replaces on the same file in one multi-command call automatically batch — all line numbers resolve against the original content, not intermediate states. No special syntax needed.

## Before You Edit

Always read the target first:

```
my-project[docs/guide.md].toc()              # Understand structure
my-project[docs/guide.md].section("Setup")   # See exact current text
```

`draft.edit()` requires exact text matching. If it fails:

1. Use `grep()` or `lines()` to find the exact current text
2. Retry with the exact text
3. Or fall back to `lines("N").draft.replace()` — no content matching needed

## Other File Operations

```
my-project[src/old.go].mv("src/new.go")     # Rename or move
my-project[src/unused.go].rm()               # Delete
```

## Reviewing and Committing

```
my-project.status()                          # Which files have drafts?
my-project.diff()                            # All changes
my-project[src/server.go].diff()             # One file's diff
my-project.commit("feat: update handler")    # Commit and push
```

## Discarding Changes

```
my-project[src/server.go].draft.discard()    # One file
my-project.restore()                          # All drafts
```