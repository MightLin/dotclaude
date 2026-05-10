---
name: sync-config
description: Sync this dotclaude repository into local Claude and Codex configuration locations. Use only inside the dotclaude repo when the user asks to sync, install, update, copy shared skills, update global Claude/Codex config, or replace old dotclaude sync workflows.
updated: 2026-05-10
version: 0.2.0
---

# Skill：同步全域設定

將此 repo 同步到 Claude / Codex 全域設定位置。repo 是此處全域 skills 的來源，但同步時不刪除目標端其他 skills。

## Changelog

### 0.2.0 - 2026-05-10
- 同步前先列出來源與目標差異。
- 若入口檔與 repo 管理的 skills 前後無差異，建議不執行同步。

## Step 1：檢查版本資訊

同步前先閱讀 root `skills/*/SKILL.md` 的 front matter 與 `## Changelog`：

- `updated`：UTC `YYYY-MM-DD`，表示 skill 最近修訂日期
- `version`：skill 版本，重大流程變更時調整
- `Changelog`：只保留最新一版，提供同步前判斷依據

版本資訊是人工判斷依據，不是自動同步邏輯。閱讀後摘要列出本次可同步的 skill 版本，詢問使用者是否繼續。

## Step 2：列出同步差異

同步前先比較 repo 來源與全域目標端，列出 `missing` / `changed` / `same` 摘要，並只針對 repo 管理的檔案比較；目標端額外存在的其他 skills 不列為刪除或差異。

比較範圍：

- Claude：`CLAUDE.md` → `$HOME\.claude\CLAUDE.md`
- Claude skills：`skills\**\*` → `$HOME\.claude\skills\**\*`
- Codex：`AGENTS.md` → `$HOME\.codex\AGENTS.md`
- Codex skills：`skills\**\*` → `$HOME\.codex\skills\**\*`

建議使用檔案 hash 比較，避免只看時間戳：

```powershell
function Compare-ManagedFile {
  param(
    [string]$Source,
    [string]$Target,
    [string]$Label
  )

  if (-not (Test-Path $Target)) {
    [pscustomobject]@{ Label = $Label; Status = "missing"; Source = $Source; Target = $Target }
    return
  }

  $sourceHash = (Get-FileHash $Source -Algorithm SHA256).Hash
  $targetHash = (Get-FileHash $Target -Algorithm SHA256).Hash
  $status = if ($sourceHash -eq $targetHash) { "same" } else { "changed" }
  [pscustomobject]@{ Label = $Label; Status = $status; Source = $Source; Target = $Target }
}

$diffs = @()
$diffs += Compare-ManagedFile "CLAUDE.md" "$HOME\.claude\CLAUDE.md" "Claude entry"
$diffs += Compare-ManagedFile "AGENTS.md" "$HOME\.codex\AGENTS.md" "Codex entry"

Get-ChildItem "skills" -File -Recurse | ForEach-Object {
  $relative = Resolve-Path $_.FullName -Relative
  $relative = $relative -replace '^\.[\\/]', ''
  $diffs += Compare-ManagedFile $_.FullName (Join-Path "$HOME\.claude" $relative) "Claude skill"
  $diffs += Compare-ManagedFile $_.FullName (Join-Path "$HOME\.codex" $relative) "Codex skill"
}

$diffs | Group-Object Status | Select-Object Name, Count
$diffs | Where-Object Status -ne "same" | Format-Table Label, Status, Source, Target -AutoSize
```

若所有項目都是 `same`，回覆使用者「前後無差異，建議不執行同步」，並停止在確認前；只有使用者明確要求強制同步時才繼續。

若有 `missing` 或 `changed`，摘要列出差異數量與主要檔案，再進入確認步驟。

## Step 3：確認同步意圖

同步會覆蓋全域入口檔與 repo 內同名 skills。執行前先摘要將同步的目標，並詢問使用者是否繼續。

- Claude：同步 `CLAUDE.md` 與 root `skills/`
- Codex：同步 `AGENTS.md` 與 root `skills/`

## Step 4：同步 Claude

覆蓋 root `skills/` 內同名檔案，但不清空 `~/.claude\skills`，避免刪除其他全域 skills。

```powershell
# CLAUDE.md
Copy-Item "CLAUDE.md" "$HOME\.claude\CLAUDE.md" -Force

# skills（全域來源）
New-Item "$HOME\.claude\skills" -ItemType Directory -Force | Out-Null
Copy-Item "skills\*" "$HOME\.claude\skills\" -Recurse -Force
```

## Step 5：同步 Codex

同步 `AGENTS.md` 與 root `skills/`。同步時不清空 `~/.codex\skills`，並保留 `.system` 與其他全域 skills。

```powershell
# AGENTS.md
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

## Step 6：確認

列出本次已同步的檔案清單。
