---
name: entrypoint-writing
description: Write concise agent entrypoint files for projects. Use when creating or updating Claude `CLAUDE.md` or Codex `AGENTS.md` files that point to `.agents/rules/`, include core working principles, and avoid duplicating detailed architecture, tech stack, TODO, or business docs.
updated: 2026-05-23
version: 0.3.0
---

## Changelog

### 0.3.0 - 2026-05-23
- 第 4 條核心原則：rules 缺口入口改為指向 `audit-rules`

### 0.2.1 - 2026-05-23
- 新增獨立的「大小上限」段，並加入超出時使用 rules-overflow skill 的提示。

### 0.2.0 - 2026-05-22
- 核心原則新增第 4 條：完成修改後主動提示 `/check-rules` 或更新 rules

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
4. 完成一段功能或大量修改後，主動建議使用者執行 `/check-rules`；若程式碼出現 `.agents/rules/` 未涵蓋的反覆模式，建議執行 `audit-rules` 審查

## 文件索引規則
- 只列實際存在的 rules 檔（依專案類型而定）
- 不列尚未建立的檔案
- 若建了 `mcp-conventions.md`，列入索引
- Claude / Codex 入口檔內容可幾乎相同，只調整標題與工具名稱

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
4. 完成修改後主動建議跑 /check-rules；發現 rules 未涵蓋的反覆模式時建議執行 audit-rules
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
4. 完成修改後主動建議跑 /check-rules；發現 rules 未涵蓋的反覆模式時建議執行 audit-rules
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
4. 完成修改後主動建議跑 /check-rules；發現 rules 未涵蓋的反覆模式時建議執行 audit-rules
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
4. 完成修改後主動建議跑 /check-rules；發現 rules 未涵蓋的反覆模式時建議執行 audit-rules
5. {專案特有規則}

## 禁止
- {容易犯的錯}
```
