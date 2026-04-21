# Claude Global Config

Claude Code 全域設定，包含 commands、skills 與行為規則，適用於所有專案。

## 目錄結構

```
~/.claude/ (AppData/Roaming/Claude/)
├── CLAUDE.md                  # 全域行為規則
├── .gitignore
├── .claude/
│   ├── commands/              # 自訂 slash commands
│   │   ├── init-project.md    # 首次導入新專案
│   │   ├── understand.md      # 全面理解當前專案
│   │   └── new-feature.md     # 開發新功能前準備流程
│   └── skills/                # 可呼叫的 skill 模組
│       ├── architecture/      # 撰寫 architecture.md
│       ├── tech-stack/        # 撰寫 tech-stack.md
│       ├── api-conventions/   # 撰寫 api-conventions.md
│       ├── design-guide/      # 撰寫 design-guide.md
│       ├── todo-and-plans/    # 撰寫 todo-and-plans.md
│       ├── project-structure/ # 建立標準專案結構
│       └── claude-md-writing/ # 撰寫精簡 CLAUDE.md
```

## Commands

| Command | 說明 |
|---|---|
| `/init-project` | 首次導入新專案，建立 `.claude/` 結構與 `CLAUDE.md` |
| `/understand` | 讀取專案 rules，全面理解當前專案 |
| `/new-feature` | 開發新功能前的標準確認流程 |

## Skills

Skills 被 commands 內部引用，提供各文件的撰寫規範（格式、必要內容、範例）。

## 使用方式

1. 進入任意專案目錄，用 Claude Code 開啟
2. 執行 `/init-project` 建立專案 rules
3. 後續每次開新對話，執行 `/understand` 讓 Claude 理解專案現況
4. 開發新功能時執行 `/new-feature`
