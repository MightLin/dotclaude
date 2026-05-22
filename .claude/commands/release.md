---
description: Bump dotclaude plugin version, commit, tag, and push to publish a new release.
---

You are releasing a new version of the **dotclaude** plugin.

## Steps

1. **Read current version** from `.claude-plugin/plugin.json` (the `version` field).

2. **Ask the user** which bump to apply, using `AskUserQuestion`:
   - patch (x.y.Z+1) — bug fix / 小修
   - minor (x.Y+1.0) — 新 skill 或新功能
   - major (X+1.0.0) — breaking change
   - Show the resulting version next to each option.

3. **Confirm working tree is clean** on `main`:
   - `git status --porcelain` should be empty
   - `git branch --show-current` should be `main`
   - If not, stop and tell the user what to fix.

4. **Update both manifest files** to the new version (keep them in sync):
   - `.claude-plugin/plugin.json` → `version`
   - `.claude-plugin/marketplace.json` → `plugins[0].version`

5. **Commit, tag, push** in one go:
   ```
   git add .claude-plugin/plugin.json .claude-plugin/marketplace.json
   git commit -m "chore(release): vX.Y.Z"
   git tag vX.Y.Z
   git push origin main
   git push origin vX.Y.Z
   ```

6. **Report** the new version and tag URL: `https://github.com/MightLin/dotclaude/releases/tag/vX.Y.Z`. Suggest the user open it on GitHub to write release notes (optional).

## Rules

- Never bump without asking — always confirm the bump type first.
- Both json files must stay on the same version. Never commit a mismatch.
- Do not run `git push --force` or skip hooks.
- If on a non-main branch, stop and ask before proceeding.
