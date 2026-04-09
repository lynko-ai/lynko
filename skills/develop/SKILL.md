---
name: lynko-develop
description: >
  Full development workflow through Lynko — read code, make edits, run tests,
  review diffs, and commit. Use when making code changes, fixing bugs, or
  implementing features in a git collection.
---

# Develop with Lynko

## Workflow Overview

1. Check for existing drafts → `status()`
2. Read relevant code → `outline()`, `expand()`, `grep()`
3. Make changes → `draft.edit()`, `draft.append()`
4. Review → `diff()`, `status()`
5. Test → `test()`
6. Commit → `commit()`

## Start of Session

Always check for pending work first:

```
my-project.status()                          # Any existing drafts?
my-project.log()                             # Recent commit history
```

## Reading Code

Navigate top-down. Never read large files in full — use structure-aware operations:

```
my-project[src/server.go].outline()        # See types, functions, signatures
my-project[src/server.go].expand("Handler") # Read one function
my-project[src/].grep("handleAuth", context_lines=3)  # Find in context
```

Use `find_definition()` and `find_references()` to trace code across files:

```
my-project.find_definition("UserService")
my-project.find_references("UserService")
```

## Editing

### Content-addressed edit (primary method)

Match existing text and replace it:

```
my-project[src/config.go].draft.edit("debug: true", "debug: false")
```

Scope to a function to avoid accidental matches elsewhere:

```
my-project[src/server.go].expand("Handler").draft.edit("old code", "new code")
```

### Multi-line edits with raw strings

Use `@@@@@` delimiters for multi-line code — no escaping needed:

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

Opening and closing delimiters must each be on their own line. The comma between arguments also goes on its own line.

If your content itself contains `@@@@@`, use an alternative delimiter: `@@@`, `"""`, or `'''`.

### Position-addressed edit (fallback)

When content matching is tricky, replace by line number:

```
my-project[src/server.go].lines("42").draft.replace("new line content")
my-project[src/server.go].lines("10-15").draft.replace("replacement block")
```

### Other operations

```
my-project[src/server.go].draft.append("\n\n// New code")  # Append to end
my-project[src/new-file.go].draft("package main\n...")     # Create new file
my-project[src/old.go].mv("src/new.go")                   # Rename/move
my-project[src/unused.go].rm()                             # Delete
```

## Edit Tips

If `draft.edit()` fails because the old text doesn't match exactly:

1. Use `grep()` or `lines()` to find the exact current text
2. Retry `draft.edit()` with the exact text
3. Or fall back to `lines("N").draft.replace()` — no content matching needed

`draft.append()` does raw concatenation with no automatic newline — include `\n` explicitly.

## Reviewing Changes

```
my-project.status()                          # File-level overview
my-project.diff()                            # All changes summarized
my-project[src/server.go].diff()          # Full diff for one file
```

## Testing

Run CI against your draft changes (no commit needed):

```
my-project.test(targets="my-component")      # Targeted test run
```

`test()` returns immediately with a run ID. Wait, then check results:

```
ci["run-20260408T1200-abc123"].ls()
ci["run-20260408T1200-abc123"].grep("FAIL|Error", context_lines=3)
```

## Committing

Review your changes, then commit:

```
my-project.diff()                            # Final review
my-project.commit("feat(auth): add token refresh")
```

Use conventional commit format: `type(scope): description`.

## Undoing Changes

```
my-project[src/server.go].draft.discard()  # Discard one file
my-project.restore()                          # Discard all drafts
```