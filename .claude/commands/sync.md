將此 repo 同步到 Claude / Codex 全域設定位置（repo 為唯一來源，目標完整覆蓋）。

## Step 1：檢查入口檔差異

執行：
```
diff CLAUDE.md ~/.claude/CLAUDE.md
diff AGENTS.md ~/.codex/AGENTS.md
```

- 若有差異，顯示 diff 內容並詢問使用者是否繼續
- 若無差異或使用者確認繼續，執行 Step 2

## Step 2：清理目標再同步 Claude

為避免目標殘留 repo 已刪除的檔案，先清空 commands/ 與 skills/ 目錄再覆蓋。

使用 PowerShell（Windows）：

```powershell
# CLAUDE.md
Copy-Item "CLAUDE.md" "$HOME\.claude\CLAUDE.md" -Force

# commands（先清後覆蓋，排除 sync.md — 此指令為 dotclaude repo 限定）
Remove-Item "$HOME\.claude\commands" -Recurse -Force -ErrorAction SilentlyContinue
New-Item "$HOME\.claude\commands" -ItemType Directory -Force | Out-Null
Get-ChildItem ".claude\commands\*" | Where-Object { $_.Name -ne "sync.md" } | Copy-Item -Destination "$HOME\.claude\commands\" -Recurse -Force

# skills（先清後覆蓋）
Remove-Item "$HOME\.claude\skills" -Recurse -Force -ErrorAction SilentlyContinue
New-Item "$HOME\.claude\skills" -ItemType Directory -Force | Out-Null
Copy-Item ".claude\skills\*" "$HOME\.claude\skills\" -Recurse -Force
```

## Step 3：同步 Codex

Codex 目前主要同步 skills；`AGENTS.md` 若使用者同意作為 Codex 全域入口，再複製到 `$HOME\.codex\AGENTS.md`。

```powershell
# AGENTS.md（可選；使用者同意時才執行）
Copy-Item "AGENTS.md" "$HOME\.codex\AGENTS.md" -Force

# skills（先清後覆蓋；保留 .system）
Get-ChildItem "$HOME\.codex\skills" -Force | Where-Object { $_.Name -ne ".system" } | Remove-Item -Recurse -Force
Copy-Item ".claude\skills\*" "$HOME\.codex\skills\" -Recurse -Force
```

不同步：`.claude/settings.local.json`、`README.md`、`.gitignore`

## Step 4：確認

列出本次已同步的檔案清單。
