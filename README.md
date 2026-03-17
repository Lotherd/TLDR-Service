# TLDR Newsletter to Slack

A Google Apps Script that automatically parses [TLDR](https://tldr.tech/) newsletters from Gmail and forwards structured summaries to a Slack channel via webhook.

## Supported Newsletters

| Newsletter | Categories |
|-----------|------------|
| **TLDR** | Big Tech & Startups, Science & Futuristic Technology, Programming Design & Data Science, Quick Links |
| **TLDR Dev** | Articles & Tutorials, Opinions & Advice, Launches & Tools, Quick Links |
| **TLDR AI** | Headlines & Launches, Deep Dives & Analysis, Engineering & Research, Quick Links |

Articles tagged as `(GitHub Repo)` across any newsletter are automatically grouped into a dedicated **GitHub Repos** section.

## How It Works

1. Searches Gmail for unread emails from TLDR (`from:tldr`)
2. Detects which newsletter variant it is (TLDR, TLDR Dev, TLDR AI)
3. Parses articles extracting: title, URL, description, and category
4. Formats a Slack message with clickable links and descriptions
5. Sends to Slack via webhook and marks the email as read
6. If Slack delivery fails, the email stays unread for retry on the next run

## Setup

### 1. Create the Google Apps Script project

1. Go to [script.google.com](https://script.google.com) and create a new project
2. Replace the default `Code.gs` content with the contents of `TLDR_Services`
3. Rename the project to something like `TLDR_Service`

### 2. Create a Slack Incoming Webhook

1. Go to [Slack API Apps](https://api.slack.com/apps) and create a new app (or use an existing one)
2. Enable **Incoming Webhooks** and create a webhook for your desired channel
3. Copy the webhook URL

### 3. Set the Slack Webhook URL

**Option A** — Via the script:

1. In `setupProperties()`, replace `"YOUR_SLACK_WEBHOOK_URL"` with your actual webhook URL
2. Run `setupProperties` from the GAS editor (select it in the function dropdown and click Run)
3. Revert the placeholder back after running (don't leave your URL in the code)

**Option B** — Via GAS UI:

1. In the GAS editor, go to **Project Settings** (gear icon)
2. Scroll to **Script Properties**
3. Add a property: key = `SLACK_WEBHOOK_URL`, value = your webhook URL

### 4. Authorize Gmail Access

1. Run `processEmails` manually once from the GAS editor
2. Google will prompt you to authorize Gmail access — accept it

### 5. Set Up a Time-Driven Trigger

1. In the GAS editor, go to **Triggers** (clock icon in the sidebar)
2. Click **Add Trigger**
3. Configure:
   - Function: `processEmails`
   - Event source: Time-driven
   - Type: Minutes timer (e.g., every 5 or 10 minutes)
4. Save

## Slack Message Format

Messages are formatted with Slack's mrkdwn syntax:

```
📩 TLDR AI 2026-03-14

*── HEADLINES & LAUNCHES ──*
  • Article Title
    _Description of the article..._

*── QUICK LINKS ──*
  • Link Title — Short description

*── GITHUB REPOS ──*
  • Repo Name
    _What this repo does..._
```

## Requirements

- A Google account with Gmail (subscribed to TLDR newsletters)
- A Slack workspace with an incoming webhook
