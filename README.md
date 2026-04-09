# Lynko

The cognitive OS for AI agents. Navigate, edit, and act on your content through a unified DSL — git repos, Google Drive, PDFs, and more.

## Quick Start

1. **Sign up** at [auth.lynko.ai](https://auth.lynko.ai/v1/ui/auth/signup) (invite required during early access)
2. **Add a data source** — connect a git repo or Google Drive folder from your dashboard
3. **Connect your AI client** — add a custom MCP connector with `https://mcp.lynko.ai/`
4. **Start exploring** — ask your agent to list files, search, or read content

Once connected, try:

```
my-project.ls()                              # See what's here
my-project[src/main.go].outline()          # Types, functions, signatures
my-project.grep("TODO", context_lines=2)   # Search across all files
```

## How It Works

Lynko exposes your content sources as **collections** — git repos, Drive folders, documents — and lets AI agents navigate them through a natural DSL. No file downloads, no format-specific tools, no context window overflow.

**Read and navigate:**

```
my-project.ls()                              # List files
my-project[README.md].toc()               # Table of contents
my-project[README.md].section("Setup")    # Read one section
my-project[src/server.go].outline()       # Code structure
my-project[src/server.go].expand("Main")  # One function's full body
```

**Search and discover:**

```
my-project.grep("handleAuth")               # Find across all files
my-project[src/].grep("TODO")               # Search under a directory
my-project[**/*.go].grep("error")           # Search only .go files
my-project.find_definition("UserService")   # Jump to definition
my-project.find_references("UserService")   # Find all usages
```

**Edit through conversation:**

```
my-project[src/config.go].draft.edit("debug: true", "debug: false")
my-project.diff()                            # Review changes
my-project.commit("fix: disable debug mode") # Commit and push
```

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
| [reference/](reference/) | DSL cheatsheet and FAQ |

## Skills

Lynko ships skills that follow the [Agent Skills](https://agentskills.io) open standard, the same format used by Claude Code, OpenAI Codex, GitHub Copilot, and Cursor.

| Skill | Use when... |
|-------|-------------|
| [explore](skills/explore/) | Browsing repos, reading docs, searching code |
| [develop](skills/develop/) | Full development workflow: read, edit, test, commit |
| [write](skills/write/) | Drafting and editing content |
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
