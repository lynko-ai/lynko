---
title: Dashboard
audience: users
last_validated: 2026-11-25
---

# Dashboard

Your dashboard is at [auth.lynko.ai/v1/ui/dashboard](https://auth.lynko.ai/v1/ui/dashboard). It's the one place you manage everything that backs your agent's view of Lynko: data sources, shared agent environments, credentials, and usage.

## What you can do

| Panel | What it's for |
|-------|---------------|
| **Getting Started** | First-run checklist, MCP connector URL, one-click add of the onboarding repo |
| **Collections** | Add and remove git repos and Google Drive folders; check sync status |
| **Pods** | Group collections and nodes into shared agent environments; manage capabilities and rotate keys |
| **Credentials** | Git PATs, Google Drive connection, SSH keys for Runner nodes |
| **Compute** | Add, edit and delete your account's SSH machines for Runner; see which pods use each |
| **Storage** | Workspace-wide artifact count, content size, and estimated context cost |
| **Usage** | Operations, tokens, and response time over 24h / 7d / 30d |
| **Roles** | Define per-connection read/write/admin scopes for a shared pod; connect at `/as/<role>` so each agent carries only the access it needs |
| **Active Sessions** | See and revoke OAuth sessions from connected AI clients |

The main dashboard groups the panels below; linked pages — a machine's add, edit and delete pages, a pod's Runner page and its Roles page — provide their own editors.

## Getting Started panel

A collapsible checklist that tracks first-run setup. Items tick green as you complete them:

1. **Account created** — done at signup
2. **Add a data source** — ticks once you have any git or Drive collection
3. **Connect an AI client** — ticks once any client has an active session
4. **Add onboarding guide** — ticks once `lynko-public` is in your workspace

The panel also shows two things you'll want at hand:

**MCP connector URL.** A copy-to-clipboard box with `https://mcp.lynko.ai/mcp`. Paste this into your AI client's custom MCP connector field — clients register automatically via dynamic client registration, so the URL is all you need.

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

Pods are shared agent environments. They group collections (data) and nodes (capabilities) into one environment for a piece of work. Each MCP connection uses exactly one pod, so switching pods changes the environment the agent is operating against.

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

Click **+ Add Node** to add another. The form lets you choose between four node types:

**CI** (`native:ci`) — no extra config. Just pick a reference name and submit. Adds `test()`.

**Skills** (`native:skills`) — no extra config. Just pick a reference name (it becomes the DSL prefix, e.g. `skills.ls()`) and submit. Surfaces every qualified `SKILL.md` across the pod's collections as one searchable, curatable set — discovery (`ls`/`search`/`info`/`read`/`status`) plus per-pod curation (`pin`/`exclude`/`include`/`reset`/`curation`). Skills are indexed from committed content; a newly committed `SKILL.md` appears after the next refresh.

**Embedding** (`native:embedding`) — no extra config. Powers semantic `search()` on the pod's collections; the index builds automatically on the first search.

**Runner** (`native:runner`) — adds `run()` and `invoke()` on your account's SSH machines (see [Compute](#compute-panel)). Choose only a **reference name**; it becomes the DSL prefix for run management, for example `runner.status()`, `runner.history()` and `runner["run-id"].lines("1-50")`. Attaching Runner ticks every machine you have at that moment for this pod; machines added later are ticked on the pod's Runner page.

The Runner row shows "M of N machines" — the machines ticked for this pod out of all of yours — and opens the pod's [Runner page](#pod-runner-page); on wide screens the row also has a **Manage** link to it. "0 of N machines" and "No machines" are warnings: the pod's agents cannot run anything yet.

Remove a node with **remove** in its row. Removing Runner detaches it from this pod only; your machines stay in Compute.

### Pod Runner page

Open it from a pod's Runner row: pod → Runner → **Manage** (or the "M of N machines" pill). It has two panels.

**Machines this pod may use** lists every machine of yours with a tick for each one this pod may use; **Save machines** saves exactly the ticked ones. The pod's agents can run only on ticked machines, and `runner.status()` lists only those. A machine that is not ticked is refused exactly like a name that does not exist, and no refusal names a machine the pod cannot use. Machines added later in Compute start unticked here. If the pod changed in another session, nothing is saved and the page shows the current ticks. Refreshing the page discards ticks you have not saved.

**Activity** is a snapshot of the pod's runs, not a live view; **Refresh** reads it again. Runs are grouped by machine and collection, each group with a meter such as "■■■■ 4 of 4 slots in use · 1 waiting". A machine's slots are how many runs of one collection the pod can have on it at once; more wait their turn, and other pods have their own. After a machine's slots are lowered, slots still held above the new number are stated apart, e.g. "· 1 above current limit (slot 3)". Each row shows the run's slot, its state (Running, Launching, Stopping, Slot reserved, Waiting or Preparing source), its kind (`Run` or `invoke · <agent>`), its run id and its age ("started 12m ago", "dispatched 20s ago" or "in line 48s"). Run ids print short — `run_` and the first 8 hex characters, such as `run_6720e56d`. Hover to see the full id, and use **Copy** to copy it: the full id is what you give an agent to read, cancel or capture that run. **Recent** lists the pod's last ten finished runs with their outcome. Runs on a machine no longer ticked appear under "Former machines". The page never shows a run's command, output or error text.

### Roles

Each pod row has a **Roles** button — scoped connection roles for that pod: named, additive read/write/admin scopes an agent can connect as, instead of your full-access `author` identity.

The roles table lists each role's capabilities, scope version, and connection URL (`…/pods/<pod-id>/as/<role>`); the implicit `author` role is shown read-only. To create a role, pick a name (1–64 chars: letters, digits, `-`, `_`; `author` and `owner` are reserved) and add grant rows:

- **Class** — `read`, `write`, or `admin`
- **Write mode** — for write grants: `none`, `append` (may only extend existing content), or `full`
- **Selector** — a specific collection, or a glob (dynamic: also matches collections added to the pod later)
- **Paths** (optional) — restrict the grant to paths within the selector

Template buttons pre-fill a role name and grant set to start from. Deleting a role requires typing its name to confirm and revokes every credential issued for it.

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

- **Upload** — give the key a name (e.g. `gpu-server`) and paste the private key body. Select that name as a machine's SSH key when adding or editing it in Compute.
- **Delete** — removes the key. Machines referencing it cannot authenticate until you restore the appropriate key or update their credential reference.

The dashboard never shows private key contents back — only the masked preview.

## Compute panel

Compute lists the SSH machines that `run()` and `invoke()` use. Machines belong to your account, not to a pod, and each pod uses only the machines ticked on its [Runner page](#pod-runner-page). The panel's intro says so: "A pod uses the machines ticked in its Runner. Slots are how many runs of one collection a pod can have on a machine at once; more wait their turn."

Each row shows the machine's name (what agents pass as `machine="..."`) with its SSH key, its connection with the work directory beneath it, its slots, its default timeout and the pods that use it — up to two by name, each linking to that pod's Runner page, then `+N more`. A machine no pod uses shows **No pods**: it runs nothing until a pod ticks it. **Edit** and the trash icon open the machine's edit and delete pages. If your machines cannot be read, the panel says "Machines are unavailable." and offers no add, edit or delete.

### Machine requirements

- **SSH access with a host key.** Lynko connects with an SSH key you upload under Credentials → SSH Keys and checks the machine's public host key; it refuses a host that presents a different key.
- **git and tmux 3.3a or newer** on the SSH account's `PATH`. The machine's add and edit pages say so under the SSH connection fields.
- **A dedicated, unprivileged account.** Every run executes as the SSH account, so use one made for Lynko (for example `lynko-runner`): not your personal account or root, no sudo, and not in the `docker` group, which is root-equivalent. Connect a dev, test or sandbox machine, never production.

### Adding and editing a machine

Click **+ Add machine**:

| Field | What to enter |
|-------|---------------|
| Name | 1–64 ASCII letters, digits, periods, underscores or hyphens, beginning with a letter or digit; unique among your machines. Agents select the machine with it: `run(..., machine="name")`. |
| Host, Port, SSH user | The target hostname or IP, its SSH port (blank means 22) and the SSH account. |
| SSH key | A key uploaded under Credentials → SSH Keys. The machine stores its name, never the private key. |
| Host key | The machine's public host key, such as `ssh-ed25519 AAAA…`. Copy it from the machine over a session you already trust, e.g. `cat /etc/ssh/ssh_host_ed25519_key.pub`. |
| Work directory | The base directory on the machine for collection working copies. |
| Default timeout | Applies when `run()` sets none. Blank means 60 minutes; otherwise one whole number of seconds, minutes or hours of at most five digits, such as `90s`, `30m` or `2h`. No fraction or composite duration such as `1h30m` is accepted here. |
| Slots per collection in each pod | 1–4; blank means 4. How many runs of one collection a pod can have on this machine at once; more wait their turn. |
| PATH prepend | One absolute directory per line, at most 64 entries and 4096 bytes per entry. Blank lines are ignored and surrounding whitespace is trimmed; order is preserved. No colon, parent-directory (..) segment or control character; no tilde, environment-variable or glob expansion. |

After you add a machine, Compute says "Machine *name* added. No pod can use it yet — tick it in a pod's Runner." A machine added later reaches no pod until you tick it on that pod's Runner page.

The edit page names the pods that use the machine. A machine's name is part of its working copies' identity, so renaming it selects different working copies; changing host, port, SSH user, host key or work directory gives the machine a fresh working copy in each pod that uses it, and the edit page warns you before you save. Commit or capture authored work in the current copies first. A run already dispatched keeps its machine definition, including PATH, even if the machine is edited, deleted or unticked.

Deleting a machine has its own page: type the machine's name to confirm. The pods that used it can no longer use it; active runs keep their saved machine definition.

A form that is out of date saves nothing and shows the current definitions; reapply your edit to them. If a save cannot be confirmed, reload and check the current definitions before trying again; the page does not retry.

## Storage panel

Workspace-wide totals across all pods:

- **Artifacts** — count of artifacts you own
- **Content Size** — total bytes pulled and stored
- **Est. Content Tokens** — approximate LLM context cost if all content were loaded. Useful for sizing what you've made available to agents.

Per-pod numbers appear inside each pod card.

## Usage panel

Operations, tokens, and response time over a chosen window. Use the **24h / 7d / 30d** buttons to switch windows.

- **Operations** — total DSL operation calls. If any failed, an `(N errors)` count appears in red next to it.
- **Tokens** — tokens processed in the window.
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