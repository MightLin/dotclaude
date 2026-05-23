---
description: Pre-flight validate, bump version, commit/tag/push, then guide submission to the Anthropic community plugin directory.
---

You are releasing a new version of the **dotclaude** plugin and walking the user through submitting it to `anthropics/claude-plugins-community`.

## Step A — Pre-flight checks (read-only, abort on any failure)

Run these in parallel where possible. If any fails, stop and tell the user exactly what to fix — do NOT proceed to bump.

1. **Branch & tree**:
   - `git branch --show-current` must be `main`
   - `git status --porcelain` must be empty

2. **Official validator** (this is the same tool reviewers use; catches manifest, frontmatter, missing-file issues):
   ```
   claude plugin validate --strict .
   ```
   Must exit 0. Surface the full output to the user on failure.

3. **Files exist at repo root**: `README.md`, `LICENSE`, `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`.

4. **plugin.json field check** — must have: `name`, `description`, `version`, `author`, `license`, `repository`. Warn (don't block) if `homepage` or `keywords` is missing.

5. **Version sync**: `plugin.json.version` must equal `marketplace.json.plugins[0].version`.

6. **Skill inventory**: count `skills/*/SKILL.md`. Print the count so the user can sanity-check nothing is missing.

## Step B — Ask which bump

Use `AskUserQuestion`. Show resulting version next to each option:
- patch (x.y.Z+1) — bug fix / 小修
- minor (x.Y+1.0) — 新 skill 或新功能
- major (X+1.0.0) — breaking change

## Step C — Bump both manifests (keep in sync)

- `.claude-plugin/plugin.json` → `version`
- `.claude-plugin/marketplace.json` → `plugins[0].version`

## Step D — Commit, tag, push

Keep the existing `vX.Y.Z` tag format (do NOT use `claude plugin tag` — it creates `{name}--v{version}` which would break our `v1.2.0`/`v1.2.1` history).

```
git add .claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "chore(release): vX.Y.Z"
git tag vX.Y.Z
git push origin main
git push origin vX.Y.Z
```

## Step E — Submission instructions

Capture the new commit SHA: `git rev-parse HEAD`. Then print to the user:

```
Released vX.Y.Z ✓
GitHub tag: https://github.com/MightLin/dotclaude/releases/tag/vX.Y.Z
  (optional: open this URL to write release notes)

To submit (or re-submit) to the Anthropic community directory:
  Form:    https://clau.de/plugin-directory-submission
  Repo:    https://github.com/MightLin/dotclaude
  Version: vX.Y.Z
  Commit:  <SHA from git rev-parse HEAD>

After submitting, run /release-status (no sooner than ~24h later —
the community catalog syncs nightly).
```

## Rules

- Never bump without asking — always confirm the bump type first.
- Never bypass `claude plugin validate --strict` failures. The whole point of pre-flight is to catch reviewer-blocking issues before submitting.
- Both json files must stay on the same version. Never commit a mismatch.
- Do not run `git push --force` or skip hooks.
- If on a non-main branch, stop and ask before proceeding.
