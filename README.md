# email-cleanup

A Claude Code skill that automatically marks low-value emails as read so your inbox only shows what matters.

## Problem

A busy inbox fills up with meeting RSVPs, auto-generated meeting notes, passing CI/CD notifications, and other noise that drowns out the handful of emails that actually need a response. Clearing it by hand every day doesn't scale, and most of that noise is safe to auto-triage.

## What it does

Fetches unread emails via the Gmail API, classifies each one as noise or keep using pattern matching (sender, subject line, snippet), and bulk-marks the noise as read -- no prompts, no confirmations for the clear cases.

**Auto-marks as read:**
- Meeting accepted / declined / tentative RSVPs
- Cancelled or updated calendar invites
- AI-generated meeting notes
- CI/CD pass notifications (not failures)
- View-only file shares
- Thread auto-replies that are just "+1", "Thanks", "LGTM", etc.

**Always keeps unread:**
- Direct emails with substantive content
- Emails where you're in the TO field with a question or action item
- Build/pipeline failures
- Emails with non-calendar attachments
- Anything from your management chain

**Asks before acting on:**
- Document shares with edit access (might be actionable)
- Emails from external senders
- Anything that doesn't clearly match a noise or keep pattern

## How it works (agent flow)

```
Gmail (unread, last N days)
   -> classify each email: noise / keep / unsure
   -> show a summary table (what's auto-marked, what's kept, what needs input)
   -> ask only about the "unsure" bucket
   -> batch-mark noise as read
   -> report counts
```

## Guardrails

- **Never touches content it isn't sure about.** Only emails matching an explicit noise pattern are auto-marked; everything else is either kept or surfaced for a decision.
- **Never deletes anything.** The only write action is removing the `UNREAD` label -- the email itself is untouched and still in the inbox.
- **Default-to-keep on ambiguity.** Anything that doesn't clearly match a noise pattern is left unread rather than guessed into the noise pile.
- **Batch writes, not blind writes.** Marking-as-read happens in one batch call per run with explicit message IDs, not a blanket "mark all as read."

## Configure

Noise patterns (senders, subject prefixes) are defined inline in `SKILL.md` and are meant to be edited for your own inbox -- swap in your own meeting-notes sender, CI system, and calendar-notification patterns.

Requires a Google Workspace CLI wrapper on `PATH` (or a local skills cache) exposing Gmail `list` and `labels remove` subcommands with JSON output.

## Usage

Invoke via Claude Code:

```
/email-cleanup
```
