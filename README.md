# Lynko

The cognitive OS for AI agents. Navigate, edit, and act on your content through a unified DSL — git repos, Google Drive, PDFs, and more.

## Quick Start

1. **Sign up** at [auth.lynko.ai](https://auth.lynko.ai/v1/ui/auth/signup) (invite required during early access)
2. **Add a data source** — connect a git repo or Google Drive folder from your dashboard
3. **Connect your AI client** — add a custom MCP connector with `https://mcp.lynko.ai/`

## What Makes It Different

Lynko gives agents **content-intelligent navigation** — not just file access, but structure-aware operations that match how content is naturally organized.

**Code:** understand structure without reading entire files.

```
my-project[src/server.go].outline()          # Types, functions, signatures
my-project[src/server.go].expand("Handler")  # One function's full body
my-project.find_definition("UserService")    # Jump to definition
my-project.find_references("UserService")    # Trace all usages
```

**Docs:** navigate by structure, not by scrolling.

```
my-project[docs/guide.md].toc()              # Table of contents
my-project[docs/guide.md].section("Setup")   # Read just one section
```

**Edit where you navigate:** the same operation that reads content also defines the edit scope.

```
my-project[src/server.go].expand("Handler").draft.edit("old", "new")   # Scoped to one function
my-project[docs/guide.md].section("Setup").draft.edit("old", "new")    # Scoped to one section
my-project.diff()                                                       # Review changes
my-project.commit("fix: update handler and docs")                       # Commit and push
```

**Smart resolution:** paths resolve automatically. `my-project[handler.go]` finds `src/internal/handler.go` if unique. `section("setup")` matches "Setup Guide". You can even skip the collection name — a bare path like `src/main.go` resolves to the right collection and runs your operation, as long as the file is unambiguous across your workspace.

Filesystem operations (`ls`, `grep`, `find`) and scoped search (`my-project[src/].grep("pattern")`) work as you'd expect. See the [DSL cheatsheet](reference/dsl-cheatsheet.md) for the full reference.

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

See [skills/README.md](skills/README.md) for installation instructions.

## Issues & Feedback

Found a bug or have a feature request? [Open an issue](../../issues/new/choose).

We're in **early access** — expect rough edges, and know that your feedback directly shapes the product. Please include your MCP client (Claude, ChatGPT, etc.) and the DSL command that failed, if applicable.

## Status

🚧 **Early Access** — Lynko is pre-launch. No SLA, but actively developed. The platform and DSL may evolve based on feedback.

## Links

- **Dashboard**: [auth.lynko.ai](https://auth.lynko.ai/v1/ui/dashboard)
- **MCP endpoint**: `https://mcp.lynko.ai/`
- **Agent Skills standard**: [agentskills.io](https://agentskills.io)
