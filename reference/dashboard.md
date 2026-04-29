---
title: Dashboard
audience: users
last_validated: 2026-04-29
---

# Dashboard

Your dashboard is at [auth.lynko.ai/v1/ui/dashboard](https://auth.lynko.ai/v1/ui/dashboard). It's the one place you manage everything that backs your agent's view of Lynko: data sources, workspaces, credentials, and usage.

## What you can do

| Panel | What it's for |
|-------|---------------|
| **Getting Started** | First-run checklist, MCP connector URL, one-click add of the onboarding repo |
| **Collections** | Add and remove git repos and Google Drive folders; check sync status |
| **Pods** | Group collections into workspaces; manage nodes (CI, Runner); rotate keys |
| **Credentials** | Git PATs, Google Drive connection, SSH keys for Runner nodes |
| **Storage** | Workspace-wide artifact count, data size, and estimated context cost |
| **Usage** | Operations, credits, and response time over 24h / 7d / 30d |
| **Active Sessions** | See and revoke OAuth sessions from connected AI clients |

The dashboard is a single page — every panel below corresponds to a section on it.

## Getting Started panel

A collapsible checklist that tracks first-run setup. Items tick green as you complete them:

1. **Account created** — done at signup
2. **Add a data source** — ticks once you have any git or Drive collection
3. **Connect an AI client** — ticks once any client has an active session
4. **Add onboarding guide** — ticks once `lynko-public` is in your workspace

The panel also shows two things you'll want at hand:

**MCP connector URL.** A copy-to-clipboard box with `https://mcp.lynko.ai/`. Paste this into your AI client's custom MCP connector field.

For Claude (Desktop and web), expand Advanced Settings and also enter:
- Client ID: `claude-browser`
- Client Secret: `claude_browser_secret`

These are public client identifiers used by the OAuth flow, not private credentials. ChatGPT, Codex, and most other MCP clients only need the URL.

**Add the onboarding repo.** A one-click **+ Add** button that adds `https://github.com/lynko-ai/lynko` to your default pod. This is the repo you're reading right now — adding it lets your agent navigate the guides, FAQ, and skills the same way it navigates your own content.

Click **Hide** at the top of the panel to collapse it once you're set up; the choice is remembered locally.

## Collections panel

Lists all data sources you own, split by type. Each row shows a short ID, source name, file count, and the number of pods it's in. Use the trash icon to delete a collection (this affects every pod that references it).

### Git Repositories

Click **+ Add Repository** and enter the clone URL. For private repos, add a PAT for that host under Credentials first (see below). You can pick a reference name — the short name agents use in DSL commands like `my-project.ls()` — or let it auto-generate from the repo name. You also pick which pods to add the repo to; the default pod (★) is pre-selected.

Each row shows:

- **Branch** — the tracked branch (defaults to `main`)
- **Size** — total bytes pulled
- **Files** — file count, or a sync badge:
  - `⟳ Syncing` while the initial pull is in progress
  - `✗ Failed` if the pull errored — call `pull()` from your agent to retry

Public repos work without a PAT. The token is only consulted on hosts where you've added one.

### Google Drive

Click **+ Add Drive Folder** and paste the folder URL or folder ID. The URL looks like `https://drive.google.com/drive/folders/...`. Shared Drives are supported — paste the shared drive folder URL the same way.

If your Google account isn't connected yet, a Google login page appears the first time. After that, all Drive folders use the same connected account.

Drive collections are **read-only through the DSL** — to edit a file, edit it in Google Drive directly. The next `pull()` picks up the change.

The same `⟳ Syncing` / `✗ Failed` badges apply. Drive collections sync asynchronously after creation, so a brand-new folder may show `⟳ Syncing` for a while before files appear.

### Reference names

Each collection has a reference name — the short identifier agents use in DSL commands. You can rename it from the Pods panel (see below). Names allow letters, digits, and `._-`, and must be unique within a pod.

## Pods panel

Pods are workspaces. They group collections (data) and nodes (capabilities) into a single context. Each MCP connection uses exactly one pod, so switching pods is how you switch your agent between projects.

Your default pod is created automatically on first visit. Create more with **+ Create Pod**.

Each pod renders as a card with these controls:

- **Pod name** — click to rename inline. Part of batch save (see below).
- **★ / ☆** — the filled star marks your default pod. Click an empty star on another pod to make it the default. New collections are pre-selected into the default pod.
- **Rotate API key** (circular arrow icon) — issues a new key and immediately revokes the old one. Use this if you suspect a key leaked. AI clients connected to this pod will need to re-authenticate.
- **Delete pod** (trash icon) — only available on non-default pods, and requires typing the pod name to confirm. Collections are not deleted; they remain in the Collections panel and can be added to another pod.
- **rev N** — the pod's revision number, used for optimistic concurrency on Save (see batch editing below).

Inside the card:

- A table of **collections in this pod** — ref name (editable), short ID, source, file count, and **remove** to detach without deleting.
- A storage line: `N artifacts · X MB · ~Y est. tokens`. The token estimate is the approximate LLM context cost if all content were loaded.
- A **Nodes** subsection (see below).
- **+ Add Collection** — opens a picker of collections you own that aren't already in this pod.
- **+ Add Node** — see below.

### Batch editing collections and ref names

Pod name and reference names are inline-editable. Click, type, and the **Save** button activates. Save submits all pending edits in one request.

Saves use **optimistic concurrency**. If another browser tab or session changed the pod since your page loaded, save fails with a "Pod changed in another session" message. Reload to see the latest, then re-apply your edits. (Unsaved edits are lost on reload — that's the trade-off for safety against silent overwrites.)

Removing a collection or node from a pod is **not** part of batch save — those are immediate destructive actions and confirm separately.

### Nodes

Nodes are services attached to a pod. Each node enables one or more DSL operations. The Nodes table shows a ref name, the underlying node ID (e.g. `native:ci`, `native:runner`), and a type (Service or Runner).

New pods come with a **CI node** by default — this is what enables `test()` against your draft content. If you don't see `test()` in `artifacts()`, the CI node isn't attached to this pod.

Click **+ Add Node** to add another. The form lets you choose between two node types:

**CI** (`native:ci`) — no extra config. Just pick a reference name and submit. Adds `test()`.

**Runner** (`native:runner`) — adds `run()` for executing arbitrary commands on a machine you've connected over SSH. Useful for builds, smoke tests, or anything that needs to happen on a specific host. The form needs:

- **Reference name** — the DSL alias. Whatever you name it becomes the prefix for run management — `runner.status()`, `runner.history()`, `runner["run-id"].read()`, etc.
- **Machine name** — a logical name. Agents pass it as `machine="..."` when a pod has multiple machines. Optional if there's only one.
- **Host**, **Port** (default 22), **SSH User**
- **SSH Credential** — pick a key you've uploaded under Credentials → SSH Keys
- **SSH Host Key** — paste the `ssh-ed25519 AAAA…` portion. Get it with `ssh-keyscan -t ed25519 <host>` from your laptop, or `cat /etc/ssh/ssh_host_ed25519_key.pub` on the target. Hostname prefix and trailing comment, if any, should be omitted.
- **Work Directory** — absolute path on the remote machine where collections are checked out
- **Default Timeout** — Go duration format (`30m`, `2h`); per-run timeouts override

Runner rows in the Nodes table are expandable — click to see the configured machines (name, host, user, credential, work dir).

Remove a node with **remove** in its row. Confirmation is shown for runners since this drops machine config.

## Credentials

Three subsections, each independent.

### Git (Personal Access Tokens)

One token per host (e.g., `github.com`, `gitlab.com`). A single token covers all your repos on that host — you don't need a separate token per repo.

- **Add** — enter host and token, click Add
- **Update** — paste a new token in the row's input and click Update
- **Revoke** — removes the token. Private repos on that host stop syncing.

Public repos don't need a token.

**Required scopes:**
- GitHub: `repo` (for private repos). Add `workflow` if you want CI integration.
- GitLab: `read_repository` (for private repos). Add `read_api` for CI.

### Google Drive

One Google account per Lynko user. Your account connects when you first add a Google Drive collection.

- **Connect** — add a Google Drive collection. If no account is connected, a Google login page appears.
- **Disconnect** — click Disconnect under Credentials. Existing Drive collections will fail until you reconnect.
- **Switch accounts** — Disconnect first, then add a new Drive collection. The login page appears for the new account.

Connecting a second Google account replaces the first. To access folders owned by another account, share those folders with your connected account in Google Drive instead.

### SSH Keys

Required only if you use Runner nodes. Lists each uploaded key by name with a masked preview and the date you added it.

- **Upload** — give the key a name (e.g. `gpu-server`) and paste the private key body. The name is what you select in the SSH Credential dropdown when adding a Runner node.
- **Delete** — removes the key. Runner nodes referencing it will fail until you upload a replacement under the same name or reconfigure the runner.

The dashboard never shows private key contents back — only the masked preview.

## Storage panel

Workspace-wide totals across all pods:

- **Artifacts** — count of artifacts you own
- **Data Size** — total bytes pulled and stored
- **Est. Cognitive Tokens** — approximate LLM context cost if all content were loaded. Useful for sizing what you've made available to agents.

Per-pod numbers appear inside each pod card.

## Usage panel

Operations, credits, and response time over a chosen window. Use the **24h / 7d / 30d** buttons to switch windows.

- **Operations** — total DSL operation calls. If any failed, an `(N errors)` count appears in red next to it.
- **Credits** — credits consumed in the window, with a usage meter underneath.
- **Response Time** — typical operation latency (when reported).
- **Per-pod table** — same metrics broken down by pod.

Usage data updates periodically, so very recent operations may not appear immediately.

## Active Sessions panel

Lists OAuth sessions for AI clients connected to your account — one row per refresh token that's still valid.

| Column | What it shows |
|--------|---------------|
| Client | The MCP client name (Claude, ChatGPT, etc.) |
| Pod | The pod the session is bound to, or `—` if user-scoped |
| Created | When the client connected |
| Expires | When the refresh token expires |
| Revoke | Immediately invalidates the session |

Revoke a session if a device was lost, a client misbehaves, or you just want to force a re-login. The client will need to re-authenticate on its next request.

## See also

- [Getting Started](../getting-started.md) — end-to-end first-run walkthrough
- [DSL cheatsheet](dsl-cheatsheet.md) — every operation, with examples
- [FAQ](faq.md) — common questions on credentials, billing, and troubleshooting