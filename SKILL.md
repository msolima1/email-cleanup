---
name: email-cleanup
description: |
  Clean up noisy inbox emails by marking them as read. Targets junk like meeting acceptance/rejection/tentative notifications, Gemini auto-generated meeting notes, calendar invitation updates, automated CI/CD notifications, and other low-value noise. Use when the user says "clean up my email", "clear email noise", "mark junk as read", "inbox cleanup", or asks to tidy their inbox.
---

# Email Cleanup

Mark noisy, low-value emails as read so the user's inbox only surfaces what matters.

## CLI Binary

```bash
GW="{{SKILL_ROOT}}/../cli-workspace/bin/cli-workspace-$(uname -s | tr '[:upper:]' '[:lower:]')-$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')"
```

## Workflow

### 1. Fetch unread emails

```bash
$GW gmail list --unread --newer-than 7d --limit 50 --output json
```

### 2. Classify each email

Categorize every unread email as **noise** or **keep**.

**Auto-noise (mark read without asking):**

| Pattern | Match on |
|---------|----------|
| Meeting accepted/declined/tentative | Subject starts with "Accepted:", "Declined:", "Tentatively accepted:", or similar RSVPs |
| Calendar updates | Subject starts with "Updated invitation:", "Canceled event:", "Invitation:" and from a known colleague (not external) |
| Gemini meeting notes | From `gemini-notes@google.com` |
| Auto-generated notifications | From `noreply@`, `no-reply@`, `notifications@`, `mailer-daemon@` with no actionable content |
| CI/CD build notifications | From GitLab/Jenkins/GitHub Actions about passing builds (not failures) |
| Google Drive sharing (view-only) | From `drive-shares-dm-noreply@google.com` where snippet says "invited you to view" |
| Thread auto-replies | Subject starts with "Re:" where user is CC'd (not TO) and content is just "+1", "Thanks", "LGTM", etc. |

**Auto-keep (never mark read):**

- Direct emails from humans with substantive content
- Emails where user is in the TO field with a question or action item
- Pipeline/build **failures**
- Emails with attachments that aren't calendar .ics files
- Anything from user's management chain

**Ask user when unsure:**
- Google Drive document shares with edit access (might be actionable)
- Emails from external senders
- Automated emails that might contain action items
- Anything that doesn't clearly fit noise or keep

### 3. Present classification

Show a summary table before taking action:

```
Auto-marking as read (X emails):
  - "Accepted: VMA Standup" from alice@geotab.com
  - "Notes: VMA Stand up" from gemini-notes@google.com
  ...

Keeping as unread (Y emails):
  - "Q1 Planning Doc" from john@geotab.com
  ...

Need your input (Z emails):
  - "Shared doc: Project Plan" from drive-shares - Mark as read?
```

Use `AskUserQuestion` for the uncertain ones, presenting them as a multi-select of emails to also mark as read.

### 4. Mark noise as read

For each noise email ID:

```bash
$GW gmail labels remove <message-id> --labels UNREAD
```

For batch processing, pipe IDs:

```bash
echo -e "id1\nid2\nid3" | $GW gmail labels remove --batch --labels UNREAD --yes
```

### 5. Report results

```
Done! Marked X emails as read.
Y emails kept unread.
```
