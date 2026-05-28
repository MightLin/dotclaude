---
name: entrypoint-writing
description: Internal sub-skill used by `init-project` and `migrate-rules` to write Claude `CLAUDE.md` / Codex `AGENTS.md` entrypoint files. Use directly only when an entrypoint already exists and needs a standalone rewrite — otherwise route through `init-project` (new projects) or `migrate-rules` (existing projects).
updated: 2026-05-29
version: 0.3.3
---

## Changelog

### 0.3.3 - 2026-05-29
- 釐清 `/check-rules` 使用時機：只在程式碼修改後或 PR 前建議執行；初始化、入口檔重建或 rules 維護後不因此要求使用者立刻執行。

### 0.3.2 - 2026-05-28
- 釐清入口檔語意：只有文件索引指向 `.agents/rules/`，核心原則與禁止事項屬入口檔本身。

### 0.3.1 - 2026-05-25
- description 改為明示「Internal sub-skill of init-project / migrate-rules」，避免使用者繞過完整流程直接呼叫

### 0.3.0 - 2026-05-23
- 第 4 條核心原則：rules 缺口入口改為指向 `audit-rules`

### 0.2.1 - 2026-05-23
- 新增獨立的「大小上限」段，並加入超出時使用 rules-overflow skill 的提示。

### 0.2.0 - 2026-05-22
- 核心原則新增第 4 條：完成作業後提示 code compliance 或 rules 維護（0.3.3 起限定 `/check-rules` 用於程式碼修改後 / PR 前）

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：撰寫精簡入口檔

## 目的
建立讓 coding agent 每次 session 都會讀到的專案入口規則。

- Claude Code 使用 `CLAUDE.md`
- Codex 使用 `AGENTS.md`
- 兩者應指向同一份 `.agents/rules/` 專案知識檔

## 核心原則
入口檔每次 session 都會被讀入，直接消耗 context。
應「小而精準」，只放每次都需要的資訊。

## 必要內容（不超過 40 行）

### 1. 專案一句話說明
### 2. 文件索引（指向實際存在的 `.agents/rules/` 檔案）
### 3. 核心原則（必含 4 條 + 專案特有規則合計 6 條以內，每次作業必遵守）
### 4. 禁止事項（容易犯錯的地方）

## 必含的 4 條核心原則

以下 4 條取代過去 `/understand`、`/new-feature` command 的功能，是 dotclaude 流程特有的，必須寫進每個導入此流程的專案入口檔（全域 CLAUDE.md / AGENTS.md 不含這些）：

1. 接到任務前，view `.agents/rules/` 下相關檔案以理解專案現況
2. 開新功能前，先讀 `todo-and-plans.md` 確認與計畫無衝突
3. 開始實作前，列出預計異動的檔案範圍給使用者確認
4. 完成程式碼修改後或 PR 前，主動建議使用者執行 `/check-rules`；若程式碼出現 `.agents/rules/` 未涵蓋的反覆模式，建議執行 `audit-rules` 審查

如果本次工作只是初始化專案、重建入口檔或維護 `.agents/rules/`，不要因此要求使用者立刻執行 `/check-rules`。Rules 本身的品質、覆蓋、過時或過肥問題，建議走 `audit-rules` → `maintain-rules`。

## 文件索引規則
- 只列實際存在的 rules 檔（依專案類型而定）
- 不列尚未建立的檔案
- 若建了 `mcp-conventions.md`，列入索引
- Claude / Codex 入口檔內容可幾乎相同，只調整標題與工具名稱
- 語意必須清楚：`.agents/rules/` 只修飾「文件索引」指向的 rule 檔；「核心原則」與「禁止事項」是入口檔自己的段落，不可寫成「`.agents/rules/` 文件索引、核心原則與禁止事項」

## 禁止放入入口檔
- 完整的功能說明或流程描述（屬 architecture.md）
- 技術棧細節（屬 tech-stack.md）
- TODO 清單（屬 todo-and-plans.md）
- 任何超過 2 行的說明

## 大小上限
入口檔不超過 40 行。
超出時請使用 rules-overflow skill 與使用者協作決定壓縮或分離。

## 範例（依專案類型）

### 純後端
```markdown
# {ProjectName}

{一句話說明專案目的}

## 文件索引
- 架構：.agents/rules/architecture.md
- 技術棧：.agents/rules/tech-stack.md
- API 規範：.agents/rules/api-conventions.md
- 資料模型：.agents/rules/data-model.md
- 業務邏輯：.agents/rules/business-logic.md
- 測試策略：.agents/rules/testing-strategy.md
- 部署：.agents/rules/deployment.md
- TODO：.agents/rules/todo-and-plans.md

## 核心原則
1. 接到任務前，view 相關 rules 檔
2. 開新功能前，先讀 todo-and-plans.md
3. 開始實作前，列出預計異動的檔案範圍
4. 完成程式碼修改後或 PR 前建議跑 /check-rules；發現 rules 未涵蓋的反覆模式時建議執行 audit-rules
5. 修改前先 view 相關檔案，不整份重寫
6. {專案特有規則}

## 禁止
- {容易犯的錯}
```

### 純前端
```markdown
# {ProjectName}

{一句話}

## 文件索引
- 架構：.agents/rules/architecture.md
- 技術棧：.agents/rules/tech-stack.md
- Design Guide：.agents/rules/design-guide.md
- API（消費端）：.agents/rules/api-conventions.md
- 業務邏輯：.agents/rules/business-logic.md
- 測試策略：.agents/rules/testing-strategy.md
- 部署：.agents/rules/deployment.md
- TODO：.agents/rules/todo-and-plans.md

## 核心原則
1. 接到任務前，view 相關 rules 檔
2. 開新功能前，先讀 todo-and-plans.md
3. 開始實作前，列出預計異動的檔案範圍
4. 完成程式碼修改後或 PR 前建議跑 /check-rules；發現 rules 未涵蓋的反覆模式時建議執行 audit-rules
5. {專案特有規則}

## 禁止
- {容易犯的錯}
```

### 全端
```markdown
# {ProjectName}

{一句話}

## 文件索引
- 架構：.agents/rules/architecture.md
- 技術棧：.agents/rules/tech-stack.md
- API 規範：.agents/rules/api-conventions.md
- 資料模型：.agents/rules/data-model.md
- Design Guide：.agents/rules/design-guide.md
- 業務邏輯：.agents/rules/business-logic.md
- 測試策略：.agents/rules/testing-strategy.md
- 部署：.agents/rules/deployment.md
- TODO：.agents/rules/todo-and-plans.md

## 核心原則
1. 接到任務前，view 相關 rules 檔
2. 開新功能前，先讀 todo-and-plans.md
3. 開始實作前，列出預計異動的檔案範圍
4. 完成程式碼修改後或 PR 前建議跑 /check-rules；發現 rules 未涵蓋的反覆模式時建議執行 audit-rules
5. {專案特有規則}

## 禁止
- {容易犯的錯}
```

### 手機 app
```markdown
# {ProjectName}

{一句話}

## 文件索引
- 架構：.agents/rules/architecture.md
- 技術棧：.agents/rules/tech-stack.md
- Design Guide：.agents/rules/design-guide.md
- API（消費端）：.agents/rules/api-conventions.md
- 資料模型（local DB）：.agents/rules/data-model.md
- 業務邏輯：.agents/rules/business-logic.md
- 測試策略：.agents/rules/testing-strategy.md
- 部署（store 上架）：.agents/rules/deployment.md
- TODO：.agents/rules/todo-and-plans.md

## 核心原則
1. 接到任務前，view 相關 rules 檔
2. 開新功能前，先讀 todo-and-plans.md
3. 開始實作前，列出預計異動的檔案範圍
4. 完成程式碼修改後或 PR 前建議跑 /check-rules；發現 rules 未涵蓋的反覆模式時建議執行 audit-rules
5. {專案特有規則}

## 禁止
- {容易犯的錯}
```
