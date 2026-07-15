---
title: Lynko
audience: users
last_validated: 2026-07-14
---

# Lynko

Connect your content once. Any connected agent can navigate your repos, docs, PDFs, and sheets through structural handles — the section, function, row, or run — not file dumps. On writable sources, authorized agents use the same surface to edit, test, and commit.

Lynko keeps shared working state addressable. Each agent assembles the context it needs from the same handles; drafts, runs, and results persist across sessions and harnesses, so the next agent resumes without rebuilding a transcript. Roles determine what an agent may do, and the server enforces those permissions on every call.

## Quick Start

1. **Sign up** at [auth.lynko.ai](https://auth.lynko.ai/v1/ui/auth/signup) (invite required during [early access](reference/early-access.md))
2. **Add a data source** — connect a git repo or Google Drive folder from your dashboard
3. **Connect your AI client** — add a custom MCP connector with `https://mcp.lynko.ai/`

## What Makes It Different

Four things, in the order you'll meet them.

**1. Handles, not file dumps.** Content-intelligent navigation — structure-aware operations that match how content is naturally organized.

Code: understand structure without reading entire files.

```
my-project[src/server.go].outline()          # Types, functions, signatures
my-project[src/server.go].expand("Handler")  # One function's full body
my-project.find_definition("UserService")    # Jump to definition
my-project.find_references("UserService")    # Trace all usages
```

Docs, PDFs, and spreadsheets: navigate by structure, not by scrolling.

```
my-project[docs/guide.md].toc()                          # Table of contents
my-project[docs/guide.md].section("Setup")               # Read one section

my-contracts[msa-2026.pdf].section("Indemnification")    # Jump to one clause in a PDF
my-finance[q1-forecast.gsheet].sheets()                  # List spreadsheet tabs
```

**2. Safe, scoped edits.** The same operation that reads content also defines the edit scope. Versioning and rollback come from git — mechanisms your team already trusts; Lynko adds the scoping, the draft workflow, and the audit trail on top.

```
my-project[src/server.go].expand("Handler").draft.edit("old", "new")   # Scoped to one function
my-project[docs/guide.md].section("Setup").draft.edit("old", "new")    # Scoped to one section
my-project.diff()                                                       # Review changes
```

**3. The full loop.** Edit → test → commit, one grammar. A CI node runs tests against your drafts before anything lands; a runner node executes commands on your machines. Your collections are the nouns; every new node adds a verb to the same grammar.

```
my-project.test(targets="api")                  # CI runs against draft state
ci["run-20260605-..."].grep("FAIL")             # Inspect results — the run is a handle too
my-project.commit("fix: update handler and docs")  # Publish
```

**4. A workspace that keeps state.** Your pod is a working copy of your content universe: drafts persist across sessions, reads see draft state, and per-pod curation (like pinning your go-to skills) shapes what agents discover — without touching the underlying repos.

**Smart resolution:** paths resolve automatically. `my-project[handler.go]` finds `src/internal/handler.go` if unique. `section("setup")` matches "Setup Guide". You can even skip the collection name — a bare path like `src/main.go` resolves to the right collection and runs your operation, as long as the file is unambiguous across your workspace.

Filesystem operations (`ls`, `grep`, `find`) and scoped search (`my-project[src/].grep("pattern")`) work as you'd expect. See the [DSL cheatsheet](reference/dsl-cheatsheet.md) for the full reference.

**Compared to a native connector:** A Drive or GitHub MCP connector moves bytes — your agent gets a snippet or a downloaded file, then has to dump it into context, build an ad-hoc parser, or guess structure from headings. Lynko exposes handles instead. The same operation that names a section, function, or row range also defines the edit scope. Bytes vs handles is the difference between "the file" and "the function you wanted to change."

## What's in This Repo

This repo is designed to be your first Lynko collection. Add `https://github.com/lynko-ai/lynko.git` from your dashboard, then explore it through conversation:

```
lynko.ls()
lynko[getting-started.md].toc()
lynko[skills/explore/SKILL.md].read()
```

The guide you're reading becomes navigable through the tool it describes.

| Path | What's inside |
|------|---------------|
| [getting-started.md](getting-started.md) | Step-by-step onboarding guide |
| [skills/](skills/) | Agent skills for common workflows |
| [reference/](reference/) | DSL cheatsheet, FAQ, and dashboard guide |

## Skills

Lynko ships skills that follow the [Agent Skills](https://agentskills.io) open standard, the same format used by Claude Code, OpenAI Codex, GitHub Copilot, and Cursor.

| Skill | Use when... |
|-------|-------------|
| [explore](skills/explore/) | Browsing repos, reading docs, searching code |
| [develop](skills/develop/) | Full development workflow: read, edit, test, commit |
| [review](skills/review/) | Reviewing changes, reading diffs |
| [run](skills/run/) | Executing commands on a target machine (builds, smoke tests, scripts) |

See [skills/README.md](skills/README.md) for installation instructions.

## Issues & Feedback

We're in **[early access](reference/early-access.md)** — expect rough edges, and your feedback directly shapes the product. We respond to everything.

- **Invite request**: fill out the [early access form](https://forms.gle/g3uk9itQ6oWRaeA98)
- **Email**: [siyuan@lynko.ai](mailto:siyuan@lynko.ai) — for urgent issues or anything you'd rather not file publicly
- **GitHub Issues**: [open an issue](../../issues/new/choose) — for bugs and feature requests. Include your MCP client (Claude, ChatGPT, etc.) and the DSL command that failed, if applicable

## Status

🚧 **[Early Access](reference/early-access.md)** — Lynko is pre-launch. No SLA, but actively developed. The platform and DSL may evolve based on feedback.

## Privacy & Data Handling

Lynko's full privacy policy is at [auth.lynko.ai/v1/ui/auth/privacy](https://auth.lynko.ai/v1/ui/auth/privacy). A short summary lives at [reference/privacy.md](reference/privacy.md) — covers what we hold, what we don't train on, and how to delete your data.

## Links

- **Dashboard**: [auth.lynko.ai](https://auth.lynko.ai/v1/ui/dashboard)
- **MCP endpoint**: `https://mcp.lynko.ai/`
- **Agent Skills standard**: [agentskills.io](https://agentskills.io)
