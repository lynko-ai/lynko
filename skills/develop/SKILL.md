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

Use multi-command syntax (multiple commands separated by newlines in one call) to reduce round trips.

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

## Merging Branches

`merge("branch")` merges the named branch INTO your current branch. Conflicts become drafts:

```
my-project.merge("main")                    # Merge main into current branch
```

The output reports auto-resolved files, conflicts, and new files. Conflicts contain `<<<<<<` / `>>>>>>` markers — resolve with normal edit operations:

```
my-project.grep("<<<<<<")                    # Find conflict markers
my-project[src/server.go].draft.edit(...)    # Resolve conflict text
my-project.diff()                            # Review the merge
my-project.commit("merge: main into feature-branch")
```

This is not a pull request — it directly merges and creates drafts. You still need `commit()` to finalize.

## Undoing Changes

```
my-project[src/server.go].draft.discard()    # Discard one file
my-project.restore()                         # Discard all drafts
```