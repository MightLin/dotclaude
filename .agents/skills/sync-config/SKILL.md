---
name: sync-config
description: Sync this dotclaude repository into local Claude and Codex configuration locations. Use only inside the dotclaude repo when the user asks to sync, install, update, copy shared skills, update global Claude/Codex config, or replace old dotclaude sync workflows.
updated: 2026-05-16
version: 0.3.0
---

# Skill：同步全域設定

將此 repo 同步到 Claude / Codex 全域設定位置。repo 是此處全域 skills 的來源，但同步時不刪除目標端其他 skills。

## Changelog

### 0.3.0 - 2026-05-16
- Step 1 同時列出來源與目標（Claude / Codex）版本，方便比對。

## Step 1：檢查版本資訊

同步前先閱讀 root `skills/*/SKILL.md` 的 front matter 與 `## Changelog`：

- `updated`：UTC `YYYY-MM-DD`，表示 skill 最近修訂日期
- `version`：skill 版本，重大流程變更時調整
- `Changelog`：只保留最新一版，提供同步前判斷依據

同步前必須同時呈現「來源」與「目標」版本，逐一讀取以下三處的 front matter：

- 來源：`skills\<name>\SKILL.md`
- Claude 目標：`$HOME\.claude\skills\<name>\SKILL.md`
- Codex 目標：`$HOME\.codex\skills\<name>\SKILL.md`

對每個 repo 管理的 skill 輸出表格，欄位：

- `skill`：skill 名稱
- `source`：repo 的 `version`（附 `updated`）
- `claude`：Claude 目標端的 `version`；檔案不存在則填 `missing`
- `codex`：Codex 目標端的 `version`；檔案不存在則填 `missing`
- `status`：`new`（任一端 missing）/ `same`（兩端與來源都相同）/ `upgrade`（來源較新）/ `downgrade`（來源較舊）/ `mixed`（Claude 與 Codex 版本不一致）

`CLAUDE.md` / `AGENTS.md` 入口檔目前沒有 front matter，版本欄位略過，差異留給 Step 2 的 hash diff 呈現。

PowerShell 範例腳本（用 regex 抓 front matter，不引入 YAML 套件）：

```powershell
function Get-SkillVersion {
  param([string]$Path)
  if (-not (Test-Path $Path)) { return $null }
  $lines = Get-Content $Path -Encoding utf8 -TotalCount 20
  $version = ($lines | Select-String '^version:\s*(.+)$').Matches.Groups[1].Value
  $updated = ($lines | Select-String '^updated:\s*(.+)$').Matches.Groups[1].Value
  [pscustomobject]@{ Version = $version; Updated = $updated }
}

function Get-Status {
  param($src, $claude, $codex)
  if (-not $claude -or -not $codex) { return "new" }
  if ($claude.Version -ne $codex.Version) { return "mixed" }
  if ($claude.Version -eq $src.Version)   { return "same" }
  # 簡單字串比較，必要時可改用 [version] 解析
  if ([version]$src.Version -gt [version]$claude.Version) { return "upgrade" }
  return "downgrade"
}

Get-ChildItem skills -Directory | ForEach-Object {
  $name   = $_.Name
  $src    = Get-SkillVersion "skills\$name\SKILL.md"
  $claude = Get-SkillVersion "$HOME\.claude\skills\$name\SKILL.md"
  $codex  = Get-SkillVersion "$HOME\.codex\skills\$name\SKILL.md"
  [pscustomobject]@{
    skill  = $name
    source = "$($src.Version) ($($src.Updated))"
    claude = if ($claude) { $claude.Version } else { "missing" }
    codex  = if ($codex)  { $codex.Version }  else { "missing" }
    status = Get-Status $src $claude $codex
  }
} | Format-Table -AutoSize
```

腳本是人工判斷依據，不是自動同步邏輯。輸出對照表後，詢問使用者是否繼續。

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
