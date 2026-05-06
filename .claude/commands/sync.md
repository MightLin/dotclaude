將此 repo 同步到 ~/.claude/（repo 為唯一來源，目標完整覆蓋）。

## Step 1：檢查 CLAUDE.md 差異

執行：
```
diff CLAUDE.md ~/.claude/CLAUDE.md
```

- 若有差異，顯示 diff 內容並詢問使用者是否繼續
- 若無差異或使用者確認繼續，執行 Step 2

## Step 2：清理目標再同步

為避免目標殘留 repo 已刪除的檔案，先清空 commands/ 與 skills/ 目錄再覆蓋：

```bash
# CLAUDE.md
cp CLAUDE.md ~/.claude/CLAUDE.md

# commands（先清後覆蓋）
rm -rf ~/.claude/commands
mkdir -p ~/.claude/commands
cp -r .claude/commands/. ~/.claude/commands/

# skills（先清後覆蓋）
rm -rf ~/.claude/skills
mkdir -p ~/.claude/skills
cp -r .claude/skills/. ~/.claude/skills/
```

不同步：`.claude/settings.local.json`、`README.md`、`.gitignore`

## Step 3：確認

列出本次已同步的檔案清單。
