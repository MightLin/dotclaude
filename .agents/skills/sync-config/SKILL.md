---
name: sync-config
description: Sync this dotclaude repository into local Claude and Codex configuration locations. Use only inside the dotclaude repo when the user asks to sync, install, update, copy shared skills, update global Claude/Codex config, or replace old dotclaude sync workflows.
updated: 2026-05-10
version: 0.1.0
---

# Skill：同步全域設定

將此 repo 同步到 Claude / Codex 全域設定位置。repo 是此處全域 skills 的來源，但同步時不刪除目標端其他 skills。

## Changelog

### 0.1.0 - 2026-05-10
- 明確化同步前參考 `updated`、`version` 與 `Changelog`，由使用者判斷是否同步。
- 同步改為覆蓋 repo 管理的檔案，不清空整個全域 skills 目錄。

## Step 1：檢查版本資訊

同步前先閱讀 root `skills/*/SKILL.md` 的 front matter 與 `## Changelog`：

- `updated`：UTC `YYYY-MM-DD`，表示 skill 最近修訂日期
- `version`：skill 版本，重大流程變更時調整
- `Changelog`：只保留最新一版，提供同步前判斷依據

版本資訊是人工判斷依據，不是自動同步邏輯。閱讀後摘要列出本次可同步的 skill 版本，詢問使用者是否繼續。

## Step 2：確認入口檔同步意圖

同步會覆蓋全域入口檔與 repo 內同名 skills。執行前先摘要將同步的目標，並詢問使用者是否繼續。

- Claude：同步 `CLAUDE.md` 與 root `skills/`
- Codex：同步 root `skills/`；`AGENTS.md` 只有在使用者同意作為 Codex 全域入口時才同步

## Step 3：同步 Claude

覆蓋 root `skills/` 內同名檔案，但不清空 `~/.claude\skills`，避免刪除其他全域 skills。

```powershell
# CLAUDE.md
Copy-Item "CLAUDE.md" "$HOME\.claude\CLAUDE.md" -Force

# skills（全域來源）
New-Item "$HOME\.claude\skills" -ItemType Directory -Force | Out-Null
Copy-Item "skills\*" "$HOME\.claude\skills\" -Recurse -Force
```

## Step 4：同步 Codex

Codex 主要同步 skills；`AGENTS.md` 若使用者同意作為 Codex 全域入口，再複製到 `$HOME\.codex\AGENTS.md`。同步時不清空 `~/.codex\skills`，並保留 `.system` 與其他全域 skills。

```powershell
# AGENTS.md（可選；使用者同意時才執行）
Copy-Item "AGENTS.md" "$HOME\.codex\AGENTS.md" -Force

# skills（覆蓋同名檔案；不刪除其他 skills）
New-Item "$HOME\.codex\skills" -ItemType Directory -Force | Out-Null
Copy-Item "skills\*" "$HOME\.codex\skills\" -Recurse -Force
```

## 不同步

- `.claude/settings.local.json`
- `README.md`
- `.gitignore`
- `.claude/agents/`（目前是 repo 內 Stitch 工作流用，尚未定義跨工具同步格式）
- `.agents/skills/sync-config/`（Codex repo-local skill，只在此 repo 內使用）
- `.claude/skills/sync-config/`（Claude Code repo-local wrapper，只在此 repo 內使用）

## Step 5：確認

列出本次已同步的檔案清單。
