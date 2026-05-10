---
name: entrypoint-writing
description: Write concise agent entrypoint files for projects. Use when creating or updating Claude `CLAUDE.md` or Codex `AGENTS.md` files that point to `.agents/rules/`, include core working principles, and avoid duplicating detailed architecture, tech stack, TODO, or business docs.
updated: 2026-05-10
version: 0.1.0
---

## Changelog

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
### 3. 核心原則（5 條以內，每次作業必遵守）
### 4. 禁止事項（容易犯錯的地方）

## 必含的 3 條核心原則

以下 3 條取代過去 `/understand`、`/new-feature` command 的功能，必須寫進每個入口檔：

1. 接到任務前，view `.agents/rules/` 下相關檔案以理解專案現況
2. 開新功能前，先讀 `todo-and-plans.md` 確認與計畫無衝突
3. 開始實作前，列出預計異動的檔案範圍給使用者確認

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
4. 修改前先 view 相關檔案，不整份重寫
5. {專案特有規則}

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
4. {專案特有規則}

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
4. {專案特有規則}

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
4. {專案特有規則}

## 禁止
- {容易犯的錯}
```
