# Agent Global Config

Claude Code / Codex 共用的全域設定，包含 skills 與行為規則，適用於所有專案。

## 目錄結構

```
repo root
├── CLAUDE.md                    # Claude 使用的 repo 入口規則
├── AGENTS.md                    # Codex 使用的 repo 入口規則
├── .mcp.json                    # MCP server 設定
├── skills/                      # 要同步到全域的 Claude / Codex skills 來源
│   ├── architecture/            # architecture.md
│   ├── tech-stack/              # tech-stack.md
│   ├── api-conventions/         # api-conventions.md
│   ├── design-guide/            # design-guide.md
│   ├── business-logic/          # business-logic.md
│   ├── data-model/              # data-model.md
│   ├── testing-strategy/        # testing-strategy.md
│   ├── deployment/              # deployment.md
│   ├── todo-and-plans/          # todo-and-plans.md
│   ├── init-project/            # 專案導入流程
│   ├── migrate-rules/           # 舊專案 rules 遷移
│   └── entrypoint-writing/      # 精簡 CLAUDE.md / AGENTS.md
├── .agents/
│   └── skills/
│       └── sync-config/         # Codex repo-local 同步流程
└── .claude/
    └── agents/                  # Claude 專用 subagents
```

## 入口方式

| 入口 | 說明 |
|---|---|
| `init-project` skill | 首次導入新專案，建立 `.agents/rules/` 與入口檔 |
| `sync-config` skill | **Codex 專用**：手動同步 repo 到 `~/.codex/skills/`（Claude Code plugin 使用者不需要） |
| `migrate-rules` skill | 舊專案從 `.claude/rules/` 遷移到 `.agents/rules/` |

過去的 `/understand`、`/new-feature` 已內化為入口檔的核心原則，每 session 自動生效，不再需要手動觸發。

## 來源分層

- `skills/`：要同步到 `~/.claude/skills/` 與 `~/.codex/skills/` 的全域 skills 來源。
- `.agents/skills/`：Codex repo-local skills；目前只放 `sync-config`。
- `.agents/rules/`：各專案產出的共用知識檔位置，不放在 `.claude/` 底下。
- `.claude/agents/`：Claude Code 專用 subagents，目前用於 Stitch 設計流程。

## Skills

全域 skills 放在 root `skills/`，作為 Claude / Codex 共用來源；同步時分別複製到 `~/.claude/skills/` 與 `~/.codex/skills/`。
它們提供各 rule 檔的撰寫規範（適用範圍、必要內容、禁止項、大小上限、範例）。

每個 `skills/*/SKILL.md` 都要在 front matter 維護版本資訊：

- `updated`：UTC `YYYY-MM-DD`，表示最近修訂日期。
- `version`：skill 版本，重大流程變更時調整。
- `## Changelog`：只保留最新一版，作為同步前的人工判斷參考。

版本資訊不作為自動同步邏輯；使用 `sync-config` 前先閱讀日期、版本與 changelog，再決定是否覆蓋到全域設定。
同步時只覆蓋 repo `skills/` 提供的同名內容，不清空全域 `skills/` 目錄，避免刪除其他來源安裝的 skills。

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
| sync-config | （同步流程） | **Codex 專用**，手動同步 repo 到全域設定；Claude Code plugin 使用者不需要 |
| migrate-rules | （遷移流程） | 已用舊版 `.claude/rules/` 的專案 |
| entrypoint-writing | CLAUDE.md / AGENTS.md | `init-project` skill 內部使用 |

## 安裝

### Claude Code Plugin（推薦）

在 Claude Code 內執行：

```
/plugin marketplace add MightLin/dotclaude
/plugin install dotclaude@dotclaude
```

安裝後 skills 以 `dotclaude:` 前綴呼叫，例如 `/dotclaude:init-project`。

更新到最新版：

```
/plugin update dotclaude@dotclaude
```

### Codex（手動同步）

1. clone 此 repo
2. 直接說「使用 sync-config skill 同步」
3. Skills 會複製到 `~/.codex/skills/`

## 使用方式

1. 進入任意專案目錄，用 Claude Code 或 Codex 開啟
2. 直接說「使用 `dotclaude:init-project` skill 初始化這個專案」
3. 專案知識檔統一放在 `.agents/rules/`，由 `CLAUDE.md` 與 `AGENTS.md` 指向
4. 已用舊版 `.claude/rules/` 的專案，使用 `dotclaude:migrate-rules` skill 遷移，不需重跑 init
