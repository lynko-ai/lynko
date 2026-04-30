---
title: Privacy & Data Handling
audience: users
last_validated: 2026-04-29
---

# Privacy & Data Handling

> Short summary. The canonical privacy policy lives at [auth.lynko.ai/v1/ui/auth/privacy](https://auth.lynko.ai/v1/ui/auth/privacy) — read that for legal terms, regulatory disclosures, and details.

## What Lynko holds

- **Account data**: your email, password (bcrypt-hashed), display name
- **Credentials**: your access tokens for connected services (git providers, Google Drive), encrypted with AES-256-GCM at rest
- **Pod manifest**: which collections you've added, configuration, ownership
- **Derived caches**: parsed structure (TOCs, outlines) for content you connect, derived from source data, recomputable from source on demand

## What Lynko does not hold

- Long-term backups of your source content. Drive files, repo content, and other source data are re-fetched from the source as needed; they're not preserved in Lynko's datastores beyond active caches.
- Email, calendar, or messaging data we haven't been asked to read.

## What we never do with your content

- **We do not use your content to train AI models.** Not for Lynko, not for any LLM provider Lynko forwards requests to, not for any third party. This is a strict policy with no carve-outs during early access.
- We do not sell your data.
- We do not share your content with advertisers.

## Encryption

- **At rest**: AES-256-GCM with derived per-pod keys (no shared encryption key across customers)
- **In transit**: TLS at the edge today; full internal-service TLS is on the near-term roadmap (tracked in our CASA Tier 2 work)
- **Auth**: RS256-signed JWTs, bcrypt-hashed passwords, refresh-token theft detection

## Deletion

- Today: account deletion is admin-initiated. Email [siyuan@lynko.ai](mailto:siyuan@lynko.ai) and we'll process within 30 days. The deletion cascades through credentials, derived caches, and pod manifest entries.
- Near-term roadmap: self-service `DELETE /v1/users/me` endpoint with the same 30-day SLA and an audit-log entry confirming the purge.

## Subprocessors

The infrastructure providers we use to operate Lynko:

- **AWS / GCP** — cloud hosting and object storage
- **Resend** — transactional email (verification, password reset, invite emails)
- **Postgres** (managed) — sacred-table storage

Updates to this list are reflected here when they happen.

## Google Drive specifically

When you connect Google Drive:

- Lynko receives an OAuth access token scoped to `drive.readonly` (read-only)
- We use this token only when you ask the agent to read content
- We adhere to Google's [Limited Use Policy](https://developers.google.com/terms/api-services-user-data-policy#additional_requirements_for_specific_api_scopes) — Drive content is used only to provide the user-facing features Lynko advertises

## Questions

Email [siyuan@lynko.ai](mailto:siyuan@lynko.ai). For security vulnerabilities specifically, follow the responsible-disclosure process in [SECURITY.md](../SECURITY.md).