---
title: Early Access
audience: users
last_validated: 2026-04-29
---

# Early Access

> Lynko is in early access while we complete Google's security verification process. Some constraints apply during this period — they all go away once verification is complete.

## What is early access?

Lynko reads from your Google Drive when you ask it to. To do that, we use Google's OAuth API. Apps that read Drive content are subject to Google's security review process (CASA Tier 2), which takes 2–3 months and requires us to operate under "Testing" mode in the meantime.

In Testing mode:

- We allowlist users one at a time after they sign up
- Users in our allowlist count against a 100-user cap (Google's limit)
- All re-authorization tokens reset every 7 days

None of this affects Lynko's git collections, MCP API, or any non-Drive functionality. It only applies to Drive integration and the period before we publish.

## Why am I being asked to sign in to Google again?

If Lynko stopped being able to read your Drive after about a week, this is why. Google requires apps in their pre-verification testing program to re-authorize all users every 7 days. The flow is identical to your original Drive connection — click through, approve, you're back.

To re-authorize:

1. Go to your dashboard at [auth.lynko.ai](https://auth.lynko.ai/v1/ui/dashboard)
2. Find your Drive collection (it'll show an "auth expired" or similar status)
3. Click reconnect, run through the Google OAuth flow as you did the first time

This will go away when we complete Google's verification.

## How do I get an invite?

Email me at [siyuan@lynko.ai](mailto:siyuan@lynko.ai) with your name and (optionally) the Google account whose Drive you want to connect. We're activating users in small batches — typical turnaround is about 24 hours.

If you signed up without sharing a Google account email and want to connect Drive later, just reply to your activation email with the address you'll use, and we'll get you set up.

## Does my Google account email need to match my Lynko account email?

**No.** They're independent.

- Your **Lynko account** uses email + password and can be any email you control. Used for signing in to the dashboard.
- Your **Google Drive account** is the Google identity that owns the Drive content you want Lynko to access. You connect it from the dashboard when you add a Drive collection.

Many users have a personal Lynko account and connect a work Google account, or vice versa. You can also connect multiple Google accounts to the same Lynko account (e.g., personal Drive + work Drive in separate collections).

## Why was my Drive access denied?

If you see a "Google hasn't verified this app" warning or an "Access blocked" page when connecting Drive, it usually means the Google account you're using isn't in our test-user allowlist yet. Email us with the Google account email you want to use, and we'll allowlist it within 24 hours.

If you've already been allowlisted but you're still seeing the warning page: yellow "this app isn't verified" warnings are expected during early access. Click "Advanced" → "Go to lynko.ai (unsafe)" to proceed. The warning goes away once Google completes our verification.

## What features aren't ready yet?

Some Drive content types currently have limited navigation:

- **`.docx` / `.xlsx` files** — Lynko can list them but can't currently navigate their structure (headings, sections, sheets, cells). For now they show only `read()`. Convert to Google Docs / Google Sheets in Drive if you need structural navigation. Native parsing is on the near-term roadmap.
- **Image-only PDFs** (scans, exported documents without text layers) — `pages()` and `read()` work, but `grep()` is not available since there's no text to search. OCR support is on the longer-term roadmap.

These limitations apply to Drive specifically. Git collections, Google Docs, Google Sheets, and PDFs with text layers all have full content-intelligent navigation.

## What if something breaks?

Reply to any email from us, or open an issue on the [public repo](https://github.com/lynko-ai/lynko/issues). We're a small team and respond to everything during early access.

## What changes after Google verification?

Once we complete CASA Tier 2:

- The 7-day re-authorization requirement goes away
- The 100-user cap goes away
- The "this app isn't verified" warning goes away
- This document goes away
- Anyone can sign up without going through our allowlist

The product itself doesn't change. The Google compliance process is the gate, not the product.