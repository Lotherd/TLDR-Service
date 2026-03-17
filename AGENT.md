# AGENT.md

This file provides guidance to Agent when working with code in this repository.

## Project Overview

Google Apps Script (GAS) service that parses TLDR newsletters (TLDR, TLDR DEV, TLDR AI) from Gmail and forwards article summaries to Slack via webhook.

## Architecture

Single-file GAS project (`TLDR_Services` — no extension, this is normal for GAS). Procedural style, no modules or build system.

**Flow**: `processEmails()` → searches Gmail for unread TLDR threads → detects newsletter type → parses articles by category → sends formatted message to Slack → marks thread as read.

**Constants** (top of file):
- `NEWSLETTER_CATEGORIES` — maps each newsletter variant to its ordered list of section headers
- `ARTICLE_PATTERN` / `GITHUB_REPO_PATTERN` / `QUICK_LINK_PATTERN` / `URL_PATTERN` — regex patterns for parsing
- `GMAIL_QUERY = "from:tldr"` — Gmail search query
- `MAX_THREADS = 20` — limits threads per run to avoid GAS quota issues

**Key functions**:
- `processEmails()` — orchestrates the pipeline: Gmail search → parse → send → mark read. Failed sends do NOT mark thread as read (automatic retry on next run).
- `detectNewsletterType(subject, body)` — determines which TLDR variant based on subject/body content. Falls back to `"TLDR"`.
- `extractUrlMap(body)` — parses all `[N] URL` reference-style links into a lookup map.
- `joinWrappedLines(lines, categories)` — merges multiline-wrapped text into single logical lines.
- `extractGitHubRepos(contentBody, urlMap)` — two-pass extraction of GitHub repo entries with descriptions.
- `parseEmail(body, newsletterType)` — three-pass parsing: (1) URL map extraction, (2) GitHub repos, (3) regular articles by category. Returns `[{ category, title, url, description }]`.
- `collectDescription(lines, startIndex)` — helper that collects description lines after an article entry.
- `buildSlackMessage(subject, articles)` — formats articles for Slack mrkdwn: grouped by category with section headers, inline links, italic descriptions.
- `sendToSlack(webhookUrl, message)` — HTTP POST to Slack webhook via `UrlFetchApp`. Throws on non-200 response.
- `getSlackWebhookUrl()` — retrieves webhook URL from `PropertiesService`. Throws if not configured.
- `setupProperties()` — one-time setup: stores Slack webhook URL in `PropertiesService`. Must be run manually from GAS editor before first use.

**Data structure** — Article object used throughout:
```javascript
{ category: String, title: String, url: String, description: String }
```

## Development

- **Runtime**: Google Apps Script (V8 engine, not Node.js). Uses GAS globals: `GmailApp`, `UrlFetchApp`, `Logger`, `PropertiesService`.
- **Deployment**: Copy/paste to script.google.com or use `clasp` CLI (`clasp push`/`clasp pull`).
- **No build, no tests, no linter** configured.
- **Entry point**: `processEmails()` — triggered manually or via GAS time-driven trigger.
- **First-time setup**: Run `setupProperties()` once from GAS editor with actual Slack webhook URL.

## Critical Notes

- Slack webhook URL is stored in `PropertiesService` (not hardcoded). Run `setupProperties()` to configure.
- WhatsApp/Twilio integration is NOT implemented — only Slack is functional.
- `twilio_2FA_recovery_code.txt` exists in project root and is in `.gitignore`.
- The file `TLDR_Services` has no `.gs` extension — GAS editor doesn't use extensions, but `clasp` expects `.js` or `.gs`.
- No deduplication — if the same newsletter is processed twice, it will post duplicates to Slack.
