# email-cleanup

A Claude Code skill that automatically marks low-value emails as read so your inbox only shows what matters.

## What it does

Fetches unread emails via the Gmail API, classifies each as noise or keep, and bulk-marks noise as read — no prompts, no confirmations.

**Auto-marks as read:**
- Meeting accepted / declined / tentative RSVPs
- Cancelled or updated calendar invites
- AI-generated meeting notes
- CI/CD pass notifications
- View-only file shares
- Auto-reply noise ("+1", "Thanks", etc.)

**Always keeps unread:**
- Direct emails with substantive content
- Emails where you're in the TO field
- Build failures
- Anything from your management chain

## Stack

- Claude Code skill (runs inside the Claude CLI)
- Gmail API via CLI workspace binary
- Pattern-based classification — no ML, no false positives

## Usage

Invoke via Claude Code:

```
/email-cleanup
```
