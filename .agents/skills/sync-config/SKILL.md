---
name: sync-config
description: Sync this dotclaude repository into local Claude and Codex configuration locations. Use only inside the dotclaude repo when the user asks to sync, install, update, copy shared skills, update global Claude/Codex config, or replace old dotclaude sync workflows.
---

# Skill：同步全域設定

將此 repo 同步到 Claude / Codex 全域設定位置。repo 是唯一來源，目標目錄完整覆蓋。

## Step 1：檢查入口檔差異

執行：

```powershell
diff CLAUDE.md ~/.claude/CLAUDE.md
diff AGENTS.md ~/.codex/AGENTS.md
```

- 若有差異，顯示 diff 內容並詢問使用者是否繼續
- 若無差異或使用者確認繼續，執行 Step 2

## Step 2：同步 Claude

為避免目標殘留 repo 已刪除的檔案，先清空 skills/ 目錄再覆蓋。

```powershell
# CLAUDE.md
Copy-Item "CLAUDE.md" "$HOME\.claude\CLAUDE.md" -Force

# skills（全域來源）
Remove-Item "$HOME\.claude\skills" -Recurse -Force -ErrorAction SilentlyContinue
New-Item "$HOME\.claude\skills" -ItemType Directory -Force | Out-Null
Copy-Item "skills\*" "$HOME\.claude\skills\" -Recurse -Force
```

## Step 3：同步 Codex

Codex 主要同步 skills；`AGENTS.md` 若使用者同意作為 Codex 全域入口，再複製到 `$HOME\.codex\AGENTS.md`。

```powershell
# AGENTS.md（可選；使用者同意時才執行）
Copy-Item "AGENTS.md" "$HOME\.codex\AGENTS.md" -Force

# skills（先清後覆蓋；保留 .system）
Get-ChildItem "$HOME\.codex\skills" -Force | Where-Object { $_.Name -ne ".system" } | Remove-Item -Recurse -Force
Copy-Item "skills\*" "$HOME\.codex\skills\" -Recurse -Force
```

## 不同步

- `.claude/settings.local.json`
- `README.md`
- `.gitignore`
- `.claude/agents/`（目前是 repo 內 Stitch 工作流用，尚未定義跨工具同步格式）
- `.agents/skills/sync-config/`（Codex repo-local skill，只在此 repo 內使用）
- `.claude/skills/sync-config/`（Claude Code repo-local wrapper，只在此 repo 內使用）

## Step 4：確認

列出本次已同步的檔案清單。
