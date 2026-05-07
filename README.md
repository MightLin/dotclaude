# Agent Global Config

Claude Code / Codex 共用的全域設定，包含 skills 與行為規則，適用於所有專案。

## 目錄結構

```
repo root
├── CLAUDE.md                    # Claude 使用的 repo 入口規則
├── AGENTS.md                    # Codex 使用的 repo 入口規則
├── .mcp.json                    # MCP server 設定
├── .agents/
│   └── skills/                  # Claude / Codex 共用 skills 來源
│       ├── architecture/        # architecture.md
│       ├── tech-stack/          # tech-stack.md
│       ├── api-conventions/     # api-conventions.md
│       ├── design-guide/        # design-guide.md
│       ├── business-logic/      # business-logic.md
│       ├── data-model/          # data-model.md
│       ├── testing-strategy/    # testing-strategy.md
│       ├── deployment/          # deployment.md
│       ├── todo-and-plans/      # todo-and-plans.md
│       ├── init-project/        # 專案導入流程
│       ├── sync-config/         # 全域設定同步流程
│       ├── migrate-rules/       # 舊專案 rules 遷移
│       ├── project-structure/   # 標準 .agents/rules 結構
│       └── entrypoint-writing/  # 精簡 CLAUDE.md / AGENTS.md
└── .claude/
    └── agents/                  # Claude 專用 subagents
```

## 入口方式

| 入口 | 說明 |
|---|---|
| `init-project` skill | 首次導入新專案，建立 `.agents/rules/` 與入口檔 |
| `sync-config` skill | 將 repo 同步到 Claude / Codex 的全域設定位置 |
| `migrate-rules` skill | 舊專案從 `.claude/rules/` 遷移到 `.agents/rules/` |

過去的 `/understand`、`/new-feature` 已內化為入口檔的核心原則，每 session 自動生效，不再需要手動觸發。

## 來源分層

- `.agents/skills/`：Claude / Codex 共用 skills 來源，所有工具都可同步使用。
- `.agents/rules/`：各專案產出的共用知識檔位置，不放在 `.claude/` 底下。
- `.claude/agents/`：Claude Code 專用 subagents，目前用於 Stitch 設計流程。

## Skills

Skills 放在 `.agents/skills/`，作為 Claude / Codex 共用來源；同步時分別複製到 `~/.claude/skills/` 與 `~/.codex/skills/`。
它們提供各 rule 檔的撰寫規範（適用範圍、必要內容、禁止項、大小上限、範例）。

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
| init-project | （導入流程） | 新專案首次建立 `.agents/rules/` 與入口檔 |
| sync-config | （同步流程） | 此 repo 同步到 Claude / Codex 全域設定位置 |
| migrate-rules | （遷移流程） | 已用舊版 `.claude/rules/` 的專案 |
| project-structure | （無 rule 檔） | `init-project` skill 內部使用 |
| entrypoint-writing | CLAUDE.md / AGENTS.md | `init-project` skill 內部使用 |

## 使用方式

1. 進入任意專案目錄，用 Claude Code 或 Codex 開啟
2. 直接說「使用 init-project skill 初始化這個專案」
3. 專案知識檔統一放在 `.agents/rules/`，由 `CLAUDE.md` 與 `AGENTS.md` 指向
4. 已用舊版 `.claude/rules/` 的專案，使用 `migrate-rules` skill 遷移，不需重跑 init
5. 修改完此 repo 後，直接說「使用 sync-config skill 同步」
