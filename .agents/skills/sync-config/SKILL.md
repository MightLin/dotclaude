---
name: sync-config
description: Sync this dotclaude repository into local Claude and Codex configuration locations. Use only inside the dotclaude repo when the user asks to sync, install, update, copy shared skills, update global Claude/Codex config, or replace old dotclaude sync workflows.
updated: 2026-05-26
version: 0.4.0
---

# Skill：同步全域設定

將此 repo 同步到 Claude / Codex 全域設定位置。repo 是此處全域 skills 的來源，但同步時不刪除目標端其他 skills。

## Changelog

### 0.4.0 - 2026-05-26
- 精簡確認步驟：從 3 次停頓改為最多 1 次（無差異時零停頓）。
- 合併版本對照表與 hash diff 至同一步驟。

## Step 1：掃描差異（自動執行，不停頓）

同時輸出版本對照表與檔案 hash diff，不詢問使用者。

**版本對照表**：對每個 repo 管理的 skill 輸出，欄位 `skill` / `source`（附 `updated`）/ `claude` / `codex` / `status`（`new` / `same` / `upgrade` / `downgrade` / `mixed`）。`CLAUDE.md` / `AGENTS.md` 無 front matter，版本欄留空，差異由 hash diff 呈現。

**hash diff**：比較範圍 `CLAUDE.md`、`AGENTS.md`、`skills/**/*`，逐一與 Claude / Codex 目標端比對，輸出 `missing` / `changed` / `same` 摘要。只比對 repo 管理的檔案，目標端多餘的 skills 不列入。

```powershell
function Get-SkillVersion {
  param([string]$Path)
  if (-not (Test-Path $Path)) { return $null }
  $lines = Get-Content $Path -Encoding utf8 -TotalCount 20
  $version = ($lines | Select-String '^version:\s*(.+)$').Matches.Groups[1].Value
  $updated = ($lines | Select-String '^updated:\s*(.+)$').Matches.Groups[1].Value
  [pscustomobject]@{ Version = $version; Updated = $updated }
}

function Get-SkillStatus {
  param($src, $claude, $codex)
  if (-not $claude -or -not $codex) { return "new" }
  if ($claude.Version -ne $codex.Version) { return "mixed" }
  if ($claude.Version -eq $src.Version)   { return "same" }
  if ([version]$src.Version -gt [version]$claude.Version) { return "upgrade" }
  return "downgrade"
}

function Compare-ManagedFile {
  param([string]$Source, [string]$Target, [string]$Label)
  if (-not (Test-Path $Target)) {
    return [pscustomobject]@{ Label = $Label; Status = "missing"; Target = $Target }
  }
  $sh = (Get-FileHash $Source -Algorithm SHA256).Hash
  $th = (Get-FileHash $Target -Algorithm SHA256).Hash
  [pscustomobject]@{ Label = $Label; Status = if ($sh -eq $th) { "same" } else { "changed" }; Target = $Target }
}

# 版本對照表
Get-ChildItem skills -Directory | ForEach-Object {
  $name = $_.Name
  $src    = Get-SkillVersion "skills\$name\SKILL.md"
  $claude = Get-SkillVersion "$HOME\.claude\skills\$name\SKILL.md"
  $codex  = Get-SkillVersion "$HOME\.codex\skills\$name\SKILL.md"
  [pscustomobject]@{
    skill  = $name
    source = "$($src.Version) ($($src.Updated))"
    claude = if ($claude) { $claude.Version } else { "missing" }
    codex  = if ($codex)  { $codex.Version }  else { "missing" }
    status = Get-SkillStatus $src $claude $codex
  }
} | Format-Table -AutoSize

# hash diff
$diffs = @()
$diffs += Compare-ManagedFile "CLAUDE.md" "$HOME\.claude\CLAUDE.md" "Claude entry"
$diffs += Compare-ManagedFile "AGENTS.md" "$HOME\.codex\AGENTS.md"  "Codex entry"
Get-ChildItem "skills" -File -Recurse | ForEach-Object {
  $rel = (Resolve-Path $_.FullName -Relative) -replace '^\.[\\/]', ''
  $diffs += Compare-ManagedFile $_.FullName (Join-Path "$HOME\.claude" $rel) "Claude skill"
  $diffs += Compare-ManagedFile $_.FullName (Join-Path "$HOME\.codex"  $rel) "Codex skill"
}
$diffs | Group-Object Status | Select-Object Name, Count
$diffs | Where-Object Status -ne "same" | Format-Table Label, Status, Target -AutoSize
```

**若全部 `same`**：直接回報「目標端已是最新，無需同步」，結束。不詢問。

**若有差異**：顯示摘要（N 個 missing / changed），進入 Step 2。

## Step 2：確認（唯一停頓點）

一句話摘要將同步的內容，詢問使用者是否繼續：

> 將同步 N 個檔案至 `~\.claude` 和 `~\.codex`，繼續嗎？

使用者確認後執行 Step 3；否則中止。

## Step 3：同步 Claude + Codex

```powershell
# Claude
Copy-Item "CLAUDE.md" "$HOME\.claude\CLAUDE.md" -Force
New-Item "$HOME\.claude\skills" -ItemType Directory -Force | Out-Null
Copy-Item "skills\*" "$HOME\.claude\skills\" -Recurse -Force

# Codex
Copy-Item "AGENTS.md" "$HOME\.codex\AGENTS.md" -Force
New-Item "$HOME\.codex\skills" -ItemType Directory -Force | Out-Null
Copy-Item "skills\*" "$HOME\.codex\skills\" -Recurse -Force
```

不清空目標端 skills 目錄，避免刪除其他全域 skills。

## Step 4：完成報告

列出本次已同步的檔案清單。

## 不同步

- `.claude/settings.local.json`
- `README.md`
- `.gitignore`
- `.claude/agents/`（repo 內 Stitch 工作流用，尚未定義跨工具同步格式）
- `.agents/skills/sync-config/`（Codex repo-local skill，只在此 repo 內使用）
- `.claude/skills/sync-config/`（Claude Code repo-local wrapper，只在此 repo 內使用）
