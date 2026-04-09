---
name: lynko-review
description: >
  Review code changes, read diffs, and cross-reference across a codebase. Use
  when reviewing pull requests, auditing changes, comparing branches, or
  verifying code quality.
---

# Review with Lynko

## Checking Current State

```
my-project.status()                          # Files with pending changes
my-project.diff()                            # Summary of all changes
my-project.log()                             # Recent commit history
my-project.log(max=20)                       # More history
my-project.log(commit="abc1234")             # Full message for a specific commit
```

## Reading Diffs

Collection-level diff gives an overview. File-level diff gives full hunks:

```
my-project.diff()                            # File names and line counts
my-project[src/server.go].diff()          # Full content diff for one file
```

## Comparing Branches

```
my-project.compare(base=main)                # Stat summary vs main
my-project.compare(base=main, mode=patch)    # Full patch vs main
```

## Understanding Changed Code

When reviewing a change, read the surrounding context:

```
my-project[src/server.go].outline()        # See the full structure
my-project[src/server.go].expand("Handler") # Read the changed function
my-project[src/server.go].lines("35-55")   # Read around the change
```

## Tracing Impact

Check where changed symbols are used:

```
my-project.find_references("Handler")        # Who calls this?
my-project.find_definition("UserService")    # Where is this defined?
my-project[src/].grep("Handler", context_lines=2)  # All mentions in src/
```

## Review Checklist

1. `status()` — what files changed?
2. `diff()` — scan the overall shape of changes
3. For each changed file:
   - `outline()` — understand the file structure
   - `diff()` on the file — read the actual changes
   - `expand()` on changed functions — see full context
4. `find_references()` on any changed function signatures — check callers
5. `grep()` for patterns that should be consistent (error handling, naming)