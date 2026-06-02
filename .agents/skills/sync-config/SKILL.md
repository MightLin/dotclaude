---
name: sync-config
description: Sync this dotclaude repository into local Claude and Codex configuration locations. Use only inside the dotclaude repo when the user asks to sync, install, update, copy shared skills, update global Claude/Codex config, or replace old dotclaude sync workflows.
updated: 2026-06-03
version: 0.6.0
---

# Skill：同步全域設定

將此 repo 同步到 Claude / Codex 全域設定位置。repo 是此處全域 skills 的來源，但同步時不刪除目標端其他 skills。

## Changelog

### 0.6.0 - 2026-06-03
- Claude 同步目標從 `~/.claude/skills/<name>/` 改為 `~/.claude/commands/<name>.md`，內容為 SKILL.md 去掉 frontmatter 後的 body，使 Claude Desktop 也能使用。
- Codex 同步邏輯不變（仍複製 skill 目錄）。

### 0.5.0 - 2026-05-27
- 新增退役 skill 偵測：讀取 `skills/.retired` 清單，在 Step 1 列出目標端仍存在的退役目錄，Step 2 一併確認，Step 3 執行刪除。

### 0.4.0 - 2026-05-26
- 精簡確認步驟：從 3 次停頓改為最多 1 次（無差異時零停頓）。
- 合併版本對照表與 hash diff 至同一步驟。

## Step 1：掃描差異（自動執行，不停頓）

同時輸出版本對照表與檔案 hash diff，不詢問使用者。

**版本對照表**：對每個 repo 管理的 skill 輸出，欄位 `skill` / `source`（附 `updated`）/ `claude`（對應 `~/.claude/commands/<name>.md`）/ `codex`（對應 `~/.codex/skills/<name>/SKILL.md`）/ `status`（`new` / `same` / `upgrade` / `downgrade` / `mixed`）。`CLAUDE.md` / `AGENTS.md` 無 front matter，版本欄留空，差異由 hash diff 呈現。

**hash diff**：
- Claude：比對 `CLAUDE.md` 與 `~/.claude/CLAUDE.md`；比對每個 `skills/<name>/SKILL.md` 的內容與 `~/.claude/commands/<name>.md` 的內容（不比 hash，因為 command 檔會去掉 skill frontmatter）。
- Codex：比對 `AGENTS.md` 與 `~/.codex/AGENTS.md`；比對 `skills/**/*` 與 `~/.codex/skills/**/*`（直接 hash 比對）。

只比對 repo 管理的檔案，目標端多餘的 commands / skills 不列入。

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

function Get-CommandBody {
  # 取出 SKILL.md 內容，去掉開頭的 YAML frontmatter（--- ... ---）
  param([string]$Path)
  $lines = Get-Content $Path -Encoding utf8 -Raw
  if ($lines -match '(?s)^---\r?\n.+?\r?\n---\r?\n(.+)$') { $Matches[1].TrimStart() }
  else { $lines }
}

# 版本對照表
Get-ChildItem skills -Directory | ForEach-Object {
  $name = $_.Name
  $src    = Get-SkillVersion "skills\$name\SKILL.md"
  $claude = Get-SkillVersion "$HOME\.claude\commands\$name.md"
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

# Claude：skill → command（比對內容字串）
Get-ChildItem "skills" -Directory | ForEach-Object {
  $name = $_.Name
  $srcPath = "skills\$name\SKILL.md"
  $cmdPath = "$HOME\.claude\commands\$name.md"
  $label   = "Claude command ($name)"
  if (-not (Test-Path $cmdPath)) {
    $diffs += [pscustomobject]@{ Label = $label; Status = "missing"; Target = $cmdPath }
  } else {
    $srcBody = Get-CommandBody $srcPath
    $cmdBody = Get-Content $cmdPath -Encoding utf8 -Raw
    $status  = if ($srcBody.Trim() -eq $cmdBody.Trim()) { "same" } else { "changed" }
    $diffs  += [pscustomobject]@{ Label = $label; Status = $status; Target = $cmdPath }
  }
}

# Codex：skill 目錄直接 hash 比對
Get-ChildItem "skills" -File -Recurse | ForEach-Object {
  $rel = (Resolve-Path $_.FullName -Relative) -replace '^\.[\\/]', ''
  $diffs += Compare-ManagedFile $_.FullName (Join-Path "$HOME\.codex" $rel) "Codex skill"
}

$diffs | Group-Object Status | Select-Object Name, Count
$diffs | Where-Object Status -ne "same" | Format-Table Label, Status, Target -AutoSize

# 退役 skill 偵測
$retiredNames = Get-Content "skills\.retired" -Encoding utf8 |
  Where-Object { $_ -notmatch '^\s*#' -and $_.Trim() -ne '' } |
  ForEach-Object { $_.Trim() }

$orphans = @()
foreach ($name in $retiredNames) {
  $claudePath = "$HOME\.claude\commands\$name.md"
  $codexPath  = "$HOME\.codex\skills\$name"
  if (Test-Path $claudePath) { $orphans += [pscustomobject]@{ Target = $claudePath } }
  if (Test-Path $codexPath)  { $orphans += [pscustomobject]@{ Target = $codexPath } }
}
if ($orphans.Count -gt 0) {
  Write-Host "`n退役 skill（仍存在於目標端，將在確認後刪除）："
  $orphans | Format-Table Target -AutoSize
}
```

**若全部 `same` 且無退役 skill**：直接回報「目標端已是最新，無需同步」，結束。不詢問。

**若有差異或有退役 skill**：顯示摘要，進入 Step 2。

## Step 2：確認（唯一停頓點）

一句話摘要將同步的內容，詢問使用者是否繼續：

> 將同步 N 個檔案並移除 M 個退役 skill 至 `~\.claude` 和 `~\.codex`，繼續嗎？

（若無退役 skill，省略「移除」部分。）使用者確認後執行 Step 3；否則中止。

## Step 3：同步 Claude + Codex

```powershell
# Claude entry
Copy-Item "CLAUDE.md" "$HOME\.claude\CLAUDE.md" -Force

# Claude commands：將每個 skill 的 SKILL.md 內容（去掉 frontmatter）寫入 ~/.claude/commands/<name>.md
New-Item "$HOME\.claude\commands" -ItemType Directory -Force | Out-Null
Get-ChildItem "skills" -Directory -Exclude ".retired" | ForEach-Object {
  $name    = $_.Name
  $srcPath = "skills\$name\SKILL.md"
  $cmdPath = "$HOME\.claude\commands\$name.md"
  $body    = Get-CommandBody $srcPath
  Set-Content $cmdPath -Value $body -Encoding utf8 -NoNewline
  Write-Host "寫入 command：$cmdPath"
}

# Codex
Copy-Item "AGENTS.md" "$HOME\.codex\AGENTS.md" -Force
New-Item "$HOME\.codex\skills" -ItemType Directory -Force | Out-Null
Get-ChildItem "skills" -Exclude ".retired" | Copy-Item -Destination "$HOME\.codex\skills\" -Recurse -Force

# 清除退役 skill
foreach ($orphan in $orphans) {
  Remove-Item $orphan.Target -Recurse -Force
  Write-Host "已刪除：$($orphan.Target)"
}
```

不清空目標端 commands / skills 目錄，避免刪除使用者自建的內容；只移除 `skills/.retired` 中明確列出的退役項目。

## Step 4：完成報告

列出本次已同步的檔案清單。

## 不同步

- `.claude/settings.local.json`
- `README.md`
- `.gitignore`
- `.claude/agents/`（repo 內 Stitch 工作流用，尚未定義跨工具同步格式）
- `.agents/skills/sync-config/`（Codex repo-local skill，只在此 repo 內使用）
- `.claude/skills/sync-config/`（Claude Code repo-local wrapper，只在此 repo 內使用）
- `skills/.retired`（退役清單，僅供 sync-config 讀取，不複製至目標端）
