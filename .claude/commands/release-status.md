---
description: Check whether dotclaude is live in the anthropics/claude-plugins-community catalog (read-only).
---

You are checking the public Anthropic community plugin catalog to see if **dotclaude** has been approved and synced.

This command is **read-only**. Never write, commit, push, or modify any files.

## Step 1 — Fetch the community catalog

Use `gh api` (works with the auth the user already has):

```
gh api repos/anthropics/claude-plugins-community/contents/.claude-plugin/marketplace.json \
  --jq '.content' | base64 -d
```

If `gh` is not authed or the call fails:
- Print the error
- Tell the user to run `gh auth status` / `gh auth login`
- Stop.

## Step 2 — Look for dotclaude

Parse the returned JSON. The catalog has a `plugins` array. An entry is "ours" if either:
- `name === "dotclaude"`, OR
- `source` (or `repository` / `source.url`) contains `MightLin/dotclaude`

## Step 3 — Read the local version

Read `.claude-plugin/plugin.json` to get the local `version`.

## Step 4 — Report one of three states

**LIVE** (matching entry found):
```
✓ dotclaude is live in the community catalog
  Catalog version: <entry.version>
  Pinned commit:   <entry.commit SHA if present>
  Local version:   <local plugin.json version>
  Catalog file:    https://github.com/anthropics/claude-plugins-community/blob/main/.claude-plugin/marketplace.json
```
If local version > catalog version, add: `⚠ A newer local release (vX.Y.Z) is not yet synced — catalog syncs nightly, can take up to 24h after approval.`

**PENDING** (no matching entry):
```
✗ dotclaude is not in the community catalog yet
Possible reasons:
  1. Still under review (no SLA published)
  2. Approved but catalog hasn't synced yet (nightly job)
  3. Submission was rejected — check the email Anthropic sent
     to the submission email address
  4. Never actually submitted — submit at:
     https://clau.de/plugin-directory-submission
```

**ERROR** (couldn't fetch / parse):
```
Could not determine status: <error message>
Try: gh auth status
     gh api repos/anthropics/claude-plugins-community
```

## Rules

- Read-only. No edits, commits, or pushes.
- Do not invent state — if the catalog fetch fails, report the failure; do not assume "pending".
