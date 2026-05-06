# Claude Global Config

Claude Code 全域設定，包含 commands、skills 與行為規則，適用於所有專案。

## 目錄結構

```
~/.claude/  (Windows: C:\Users\<username>\.claude\)
├── CLAUDE.md                    # 全域行為規則
├── .gitignore
└── .claude/
    ├── commands/                # 自訂 slash commands
    │   ├── init-project.md      # 首次導入新專案
    │   └── sync.md              # 同步 repo 到 ~/.claude/
    └── skills/                  # 撰寫各 rule 檔的規範
        ├── architecture/        # architecture.md
        ├── tech-stack/          # tech-stack.md
        ├── api-conventions/     # api-conventions.md
        ├── design-guide/        # design-guide.md
        ├── business-logic/      # business-logic.md
        ├── data-model/          # data-model.md
        ├── testing-strategy/    # testing-strategy.md
        ├── deployment/          # deployment.md
        ├── todo-and-plans/      # todo-and-plans.md
        ├── project-structure/   # 標準 .claude/ 結構
        └── claude-md-writing/   # 精簡 CLAUDE.md
```

## Commands

| Command | 說明 |
|---|---|
| `/init-project` | 首次導入新專案，偵測類型後條件式建立 `.claude/` 結構與 `CLAUDE.md` |
| `/sync` | 將 repo 同步到 `~/.claude/`（repo 為唯一來源，目標完整覆蓋） |

過去的 `/understand`、`/new-feature` 已內化為 `CLAUDE.md` 的核心原則，每 session 自動生效，不再需要手動觸發。

## Skills

Skills 提供各 rule 檔的撰寫規範（適用範圍、必要內容、禁止項、大小上限、範例）。

| Skill | 對應 rule 檔 | 適用 |
|---|---|---|
| architecture | architecture.md | 所有類型 |
| tech-stack | tech-stack.md | 所有類型（含 frontend/backend/mobile 條件分支） |
| business-logic | business-logic.md | 有領域邏輯的專案 |
| todo-and-plans | todo-and-plans.md | 未用 issue tracker 的專案 |
| testing-strategy | testing-strategy.md | 所有類型 |
| deployment | deployment.md | 所有類型（含手機 store 上架） |
| api-conventions | api-conventions.md | 後端 / 全端必要；前端 / 手機條件 |
| design-guide | design-guide.md | 前端 / 全端 / 手機（含 web/mobile 條件分支） |
| data-model | data-model.md | 後端 / 全端必要；手機有 local DB 才建 |
| project-structure | （無 rule 檔） | `/init-project` 內部使用 |
| claude-md-writing | CLAUDE.md | `/init-project` 內部使用 |

## 使用方式

1. 進入任意專案目錄，用 Claude Code 開啟
2. 執行 `/init-project` 偵測專案類型並建立對應 rules
3. 後續開發中，CLAUDE.md 會引導 Claude 自動讀取相關 rules、確認 todo、列出異動範圍
4. 修改完此 repo 後執行 `/sync` 同步到 `~/.claude/`
