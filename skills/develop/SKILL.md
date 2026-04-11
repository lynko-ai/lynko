---
name: lynko-develop
description: >
  Full development workflow through Lynko — read code, make edits, run tests,
  review diffs, and commit. Use when making code changes, fixing bugs, or
  implementing features in a git collection.
---

# Develop with Lynko

For full syntax details, see the [DSL cheatsheet](../reference/dsl-cheatsheet.md).

## Workflow

1. **Check state** → `status()`, `log()`
2. **Read code** → `outline()`, `expand()`, `grep()`
3. **Edit** → `draft.edit()`, `lines().draft.replace()`
4. **Review** → `diff()`, `status()`
5. **Test** → `test()`, `ci["run-ID"].ls()`
6. **Commit** → `commit()`

## 1. Check State

```
my-project.status()                          # Any pending drafts?
my-project.log()                             # Recent commits
```

## 2. Read Code

Navigate top-down — never read large files in full:

```
my-project[src/server.go].outline()          # Types, functions, signatures
my-project[src/server.go].expand("Handler")  # One function's body
my-project.find_definition("UserService")    # Jump to definition
my-project.find_references("UserService")    # Trace all usages
my-project[src/].grep("handleAuth", context_lines=3)
```

## 3. Edit

**Content-addressed** (primary) — match and replace exact text:

```
my-project[src/server.go].expand("Handler").draft.edit("old code", "new code")
```

**Position-addressed** (fallback) — replace by line number:

```
my-project[src/server.go].lines("42").draft.replace("new content")
```

**New files:**

```
my-project[src/new-file.go].draft("package main\n...")
```

## 4. Review

```
my-project.status()                          # Which files changed?
my-project.diff()                            # All changes summarized
my-project[src/server.go].diff()             # One file's full diff
```

## 5. Test

`test()` runs against draft content — no commit needed:

```
my-project.test(targets="my-component")
```

Navigate results top-down:

```
ci.history()                                 # Recent runs with status
ci["run-ID"].ls()                            # Results: passed/failed/skipped
ci["run-ID/34"].read()                       # Drill into one result
ci["run-ID"].grep("FAIL", context_lines=3)   # Search across results
```

## 6. Commit

```
my-project.diff()                            # Final review
my-project.commit("feat(auth): add token refresh")
```

## Undoing Changes

```
my-project[src/server.go].draft.discard()    # Discard one file
my-project.restore()                         # Discard all drafts
```
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

Run tests against your **draft content** — no commit needed. Always test before committing:

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