# Getting Started with Lynko

A step-by-step guide from zero to your first edit.

## Prerequisites

- An invite code (early access)
- An MCP-compatible AI client (Claude, ChatGPT, or any client supporting custom MCP connectors)

## 1. Create Your Account

1. Go to [auth.lynko.ai/v1/ui/auth/signup](https://auth.lynko.ai/v1/ui/auth/signup)
2. Enter your invite code, email, and display name
3. Agree to the terms of service
4. Verify your email — you'll be redirected to the dashboard

## 2. Add a Data Source

From your dashboard, add a collection:

**Git repository:**
- Click **+ Add Git Repository**
- Enter the clone URL (e.g., `https://github.com/you/your-repo.git`)
- For private repos, add a personal access token under Credentials first
- The repo syncs automatically

**Google Drive:**
- Click **+ Add Google Drive**
- Authorize access to your Google account
- Select the folder to connect

Your collections appear in the Collections panel once synced.

## 3. Connect Your AI Client

Add a custom MCP connector in your AI client with this URL:

**URL:** `https://mcp.lynko.ai/`

**Most MCP clients (ChatGPT, Codex, etc.):** just the URL is enough. These clients handle registration automatically.

**Claude Desktop / Claude.ai:** Claude requires a Client ID and Client Secret because it doesn't support automatic registration. Under Advanced Settings, set:
- Client ID: `claude-browser`
- Client Secret: `claude_browser_secret`

These are public client identifiers for the OAuth flow, not private credentials. Your account security relies on the OAuth login, not these values.

Tested with Claude Desktop, Claude.ai, and ChatGPT. Other MCP-compatible clients should work — [let us know](../../issues/new/choose) if you hit issues.

Once connected, your agent has access to all collections in your workspace.

## 4. Core Concepts

**Collections** are data sources — a git repo, a Drive folder, a document. Each collection has a reference name you use in commands.

**Pods** are workspaces that group collections together. Your default pod is created automatically. You can create additional pods to organize different projects.

**The DSL** is how agents interact with your content. Operations like `ls()`, `read()`, `grep()` let agents navigate without downloading entire files.

**Drafts** are uncommitted changes. When you edit through Lynko, changes are staged as drafts. Use `diff()` to review and `commit()` to save.

## 5. Your First Session

If you're reading this guide through Lynko, you've already completed the most important step — you added this repo as a collection. If not, add `https://github.com/lynko-ai/lynko.git` from your dashboard now. It's the best way to learn Lynko: the guide you're reading becomes navigable through the tool it describes.

Start by seeing what's available:

```
artifacts()                                  # List all collections and operations
lynko[getting-started.md].toc()              # Navigate this guide through Lynko
```

Then explore a collection:

```
my-project.ls()                              # What files are here?
my-project.ls("src/")                        # What's in src/?
my-project[README.md].read()              # Read the README
```

Navigate structured content:

```
my-project[docs/guide.md].toc()           # See the table of contents
my-project[docs/guide.md].section("Setup") # Read just the Setup section
```

Navigate code:

```
my-project[src/main.go].outline()          # See types, functions, signatures
my-project[src/main.go].expand("Handler")  # Read one function in full
```

Search across files:

```
my-project.grep("TODO")                      # Search everything
my-project[src/].grep("handleAuth")          # Search under src/
my-project[**/*.go].grep("error")            # Search only .go files
```

## 6. Your First Edit

Make a change:

```
my-project[README.md].draft.edit("old text here", "new text here")
```

Review what changed:

```
my-project.status()                          # Which files have drafts?
my-project.diff()                            # See the actual changes
my-project[README.md].diff()              # Diff for one file
```

Commit when ready:

```
my-project.commit("docs: update README")
```

If you want to undo:

```
my-project[README.md].draft.discard()     # Discard one file's draft
my-project.restore()                         # Discard all drafts
```

## 7. Tips

- **Use multi-command syntax** — put multiple commands in one call, separated by newlines. This is faster and uses fewer round trips.
- **Navigate before reading** — use `toc()`, `outline()`, and `ls()` to find what you need before loading full content. This keeps context usage minimal.
- **Scope your searches** — `my-project[src/].grep("pattern")` is faster and more relevant than searching the entire repo.
- **Check status first** — run `status()` at the start of a session to see if there are pending drafts from a previous session.
- **Use partial paths** — `my-project["handler.go"]` auto-resolves to `src/internal/handler.go` if the filename is unique.

## Next Steps

- Browse the [DSL cheatsheet](reference/dsl-cheatsheet.md) for all operations
- Load a [skill](skills/) for your workflow
- Read the [FAQ](reference/faq.md) for common questions