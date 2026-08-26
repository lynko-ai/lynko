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
2. **Read + trace** → `outline()`, `expand()`, `find_references()`, `grep()`
3. **Minimize the design** → prove the live surface before adding or hardening abstractions
4. **Edit** → `draft.edit()`, `lines().draft.replace()`
5. **Review + truth sweep** → `diff()`, `status()`, references + grep-zero checks
6. **Test** → `test()`, `ci["run-ID"].ls()`
7. **Commit** → `commit()`

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

## 3. Minimize the Design

Before editing, trace the production call graph for every API/type/helper you plan to add, widen, or repair.

- **Zero production callers:** prefer deletion over hardening dead surface.
- **One specialized caller:** prefer a specialized contract; preserve generality only for concrete competing callers.
- Before adding validation, adapters, outcomes, or tests around an awkward abstraction, ask whether deleting or narrowing it removes the defect entirely.
- Trace the contract through adjacent wrappers/layers so a narrowed lower-level API is not still advertised generically above it.
- Treat absence as a claim: verify "no callers" with both reference tracing and a broad textual sweep.

For persistence, initialization, backfill/migration, deployment, or concurrency changes, write down the reachable states and transitions before coding. Identify the **single authority** for each durable transition and check fresh install, upgrade, retry/interruption, partial deployment, and concurrent writers where applicable. Prefer idempotent or fail-closed transitions over process-local heuristics.

## 4. Edit

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

## 5. Review + Truth Sweep

```
my-project.status()                          # Which files changed?
my-project.diff()                            # All changes summarized
my-project[src/server.go].diff()             # One file's full diff
```

Before testing/commit:

- Re-run `find_references()` for deleted or narrowed symbols and grep retired vocabulary across the relevant tree.
- When changing one member of a paired/enumerated contract, reread the whole set; a correct local diff can leave its sibling stale.
- Recheck comments/docs/tests at the adjacent abstraction layers so the code and advertised contract still say the same thing.
- For load-bearing claims such as "all writers", "sealed", "deadlock-free", or "never", identify the invariant that makes the bad state impossible; a passing example alone is not proof.

## 6. Test

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

## 7. Commit

```
my-project.diff()                            # Final review
my-project.commit("feat(auth): add token refresh")
```

`commit()` pushes as part of the call. If it errors but `log()` shows your commit, the push half failed (remote-side): do not re-commit — reads will serve pre-commit content and a re-commit of the same paths is refused as base-conflicted, while `pull()` won't help (origin is behind, not ahead). Run `push()` to complete it.

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