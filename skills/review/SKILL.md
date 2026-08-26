---
name: lynko-review
description: >
  Review code changes, read diffs, and cross-reference across a codebase. Use
  when reviewing pull requests, auditing changes, comparing branches, or
  verifying code quality.
---

# Review with Lynko

> **Dispatch a reviewer.** You can hand a review off to a sub-agent with `invoke()` — e.g. `my-project.invoke("claude-code", "reviewer", "Review the diff on this branch")`. The sub-agent runs attenuated to the `reviewer` role (no more powerful than your connection), non-chaining and run-bounded, and you read its findings through `runner["run-ID"].read()`. See the [DSL cheatsheet](../../reference/dsl-cheatsheet.md#invoking-agents). The risk tiers below still apply: a dispatched reviewer must not auto-approve Tier 3 changes.

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
3. **Architecture sanity / degrowth pass** — trace live callers before hardening abstractions
4. **Classify risk tier** (see below) — path and content signals decide depth
5. Review each file to tier-appropriate depth
6. Report findings grouped by tier; never mark Tier 3 as approved without human review

### Architecture sanity / degrowth pass

For every meaningful changed surface, identify its production callers and adjacent wrappers before judging the patch locally.

- Zero production callers → first evaluate deletion.
- One specialized caller → first evaluate narrowing to that caller's real contract.
- Prefer **deletion → narrowing → reuse → new abstraction** unless concrete callers require broader machinery.
- If a fix adds validators, adapters, outcome types, helper layers, or tests, ask whether removing an obsolete capability makes that machinery unnecessary.
- After narrowing a contract, trace upward and downward until every layer advertises the same semantic boundary.
- "No callers", "all writers", "complete list", and similar absence/completeness statements are claims: verify with both reference tracing and a broad textual sweep.

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

- Required symbol-first flow before broad grep on the risky path: `outline()` only the changed or conflict-relevant file(s), `expand()` each changed production symbol involved, `find_references()` for changed signatures or behavior-affecting symbols, then `grep()` for legacy patterns and uncaught dynamic usage.
- Example: `my-project[src/server.go].outline()` → `my-project[src/server.go].expand("Handler")` → `my-project.find_references("Handler")` → `my-project[src/].grep("legacyHandler", context_lines=2)`. Do not outline unrelated changed files unless the trace leads there.
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

### Tier 3+ — owner decisions, outside the review's authority

Some findings are not changes to approve or reject — they are decisions reserved to the plan's owner. A review may surface, price, and recommend; it may not rule. Escalate rather than decide when a finding implies:

- **Scope change:** reducing, closing, splitting, or re-sequencing a plan's phases.
- **Gate change:** setting, raising, or waiving a budget cap, tripwire, or stated stop.
- **Criteria change:** treating a phase as done while its success criteria are unmet.
- **Contract change:** altering approved invariants, wire shapes, or refusal semantics.
- **Record change:** marking issues resolved, or closing a carried obligation that was not discharged.

Report as **"OWNER DECISION — <the decision>, options and prices"**, and leave the plan's state open. A review that resolves one of these has exceeded its authority even when its reasoning is correct.

**Silence does not delegate.** An unanswered question, an interrupt, or "continue" authorizes further work — never a decision in this class. Where the owner has not ruled, do the work that does not depend on the ruling and re-surface the question; never pick the option that unblocks the review.

### Classification caveat — path beats content

If a change appears mechanical but touches a Tier 3 path (a "formatting" change in `auth/middleware.go`, a "comment update" in a migration file), treat as Tier 3. **Path signal overrides content signal.** Agents reviewing their own previous edits are the most vulnerable to this — a change that looks cosmetic on diff may have non-cosmetic effect in a security-sensitive path. When in doubt, escalate.