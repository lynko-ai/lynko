---
title: Dashboard & Credentials
audience: users
last_validated: 2026-04-23
---

# Dashboard & Credentials

Your dashboard is at [auth.lynko.ai/v1/ui/dashboard](https://auth.lynko.ai/v1/ui/dashboard). From here you manage collections, pods, and credentials.

## Credentials

Manage credentials under the **Credentials** section of your dashboard.

### Git (Personal Access Tokens)

One token per git host (e.g., `github.com`, `gitlab.com`). A single token covers all your repos on that host — you don't need a separate token per repo.

- **Add:** Enter the host and token, click Add
- **Update:** Paste a new token and click Update
- **Revoke:** Removes the token. Private repos on that host will stop syncing.

Public repos don't need a token.

**Required scopes:**
- GitHub: `repo` (for private repos). Add `workflow` if you want CI integration.
- GitLab: `read_repository` (for private repos). Add `read_api` for CI.

### Google Drive

One Google account per Lynko user. Your account is connected when you first add a Google Drive collection.

- **Connect:** Add a Google Drive collection — if no account is connected, a Google login page appears
- **Disconnect:** Click Disconnect under Credentials. Existing Drive collections will fail until you reconnect.
- **Switch accounts:** Disconnect first, then add a new Drive collection — the login page will appear for the new account.

Connecting a second Google account replaces the first. To access folders from another account, use Google Drive sharing to share those folders with your connected account.

## Collections

### Adding a Git Repository

Click **+ Add Git Repository** and enter the clone URL. For private repos, add a PAT for that host under Credentials first. Choose a reference name (the short name you'll use in DSL commands) and select which pods to add it to.

### Adding a Google Drive Folder

Click **+ Add Google Drive** and paste the folder URL or folder ID. The folder URL looks like `https://drive.google.com/drive/folders/...`. Shared Drives are supported — paste the shared drive folder URL the same way.

Drive collections are read-only through the DSL. To edit files, use Google Drive directly.

### Reference Names

Each collection gets a reference name — the short name used in DSL commands (e.g., `my-project.ls()`). You can set a custom name or let it auto-generate. Names should be short, descriptive, and unique within your workspace.

## Pods

Pods group collections into workspaces. Your default pod is created automatically. You can create additional pods to organize different projects or contexts.

- Each MCP connection uses one pod
- Collections can be in multiple pods
- Drafts are scoped to the pod — edits in one pod don't affect another

Each pod shows its attached **nodes** (e.g., `ci → native:ci`) on its card. Nodes control which services are available — for example, `test()` only works when a CI node is attached. New pods come with a CI node by default.