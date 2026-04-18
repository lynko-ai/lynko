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

## Comparing and Merging Branches

```
my-project.compare(base=main)                # Stat summary vs main
my-project.compare(base=main, mode=patch)    # Full patch vs main
my-project.merge("main")                     # Merge main into current branch
```

`merge()` creates drafts for all affected files. Conflicts have `<<<<<<` markers — resolve with `draft.edit()`, then `commit()`.

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
3. **Classify risk tier** (see below) — path and content signals decide depth
4. Review each file to tier-appropriate depth
5. Report findings grouped by tier; never mark Tier 3 as approved without human review

## Risk Tiers

Review is not uniform. Mechanical violations route back to the agent loop cheaply; judgment-bound changes must surface to a human and never be auto-approved. Classify before reviewing.

### Tier 3 — Judgment required, do not auto-approve

Detect by path or content signal:

- **Database migrations:** `migrations/`, `schema/`, `*.sql`; diffs containing `ALTER TABLE`, `DROP`, `CREATE INDEX`
- **Auth / permissions:** `auth/`, `permissions/`, `iam/`, auth-layer middleware, changes to `@requires_*` decorators or equivalent
- **External contracts:** public API routes, SDK / proto files, OpenAPI / GraphQL schemas
- **Dependencies:** `package.json`, `go.mod`, `requirements.txt`, `Cargo.toml`, `pyproject.toml` — any added or upgraded third-party package
- **Security-sensitive:** encryption, key / secret handling, session management, CSRF / CORS config
- **Production reliability:** retry / timeout / circuit-breaker logic, rate limits, resource limits, feature flags gating production behavior
- **Infrastructure and CI:** `*.tf`, Kubernetes manifests, `.github/workflows/`, deploy configs

For Tier 3:

- Full review: `outline()`, `diff()`, `expand()` on changed symbols, `find_references()` on changed signatures
- `grep()` the broader codebase for callers and prior patterns
- Report format: **"TIER 3 — human judgment required: <specific reason>"**
- Do not write "LGTM" — write "review complete, human approval required before merge"

### Tier 2 — Standard review

Default for most feature work, bug fixes, and refactors that don't touch Tier 3 surfaces.

- `outline()` + file-level `diff()` per changed file
- `expand()` on changed functions
- `find_references()` on changed signatures
- `grep()` for consistency with established patterns (error handling, naming, logging)

### Tier 1 — Light review

Low-blast-radius mechanical changes:

- Whitespace / formatting-only diffs
- Comment-only changes
- Test-file-only additions (new tests, no production code changes)
- Documentation (`*.md`, `docs/`)

`status()` + `diff()` sufficient. Skip the full checklist unless the diff reveals surprises.

### Classification caveat — path beats content

If a change appears mechanical but touches a Tier 3 path (a "formatting" change in `auth/middleware.go`, a "comment update" in a migration file), treat as Tier 3. **Path signal overrides content signal.** Agents reviewing their own previous edits are the most vulnerable to this — a change that looks cosmetic on diff may have non-cosmetic effect in a security-sensitive path. When in doubt, escalate.