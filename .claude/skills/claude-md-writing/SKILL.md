# Skill：撰寫精簡的 CLAUDE.md

## 核心原則
CLAUDE.md 每次 session 都會被讀入，直接消耗 token。
應「小而精準」，只放每次都需要的資訊。

## 必要內容（不超過 40 行）

### 1. 專案一句話說明
### 2. 文件索引（指向實際存在的 .claude/rules/ 檔案）
### 3. 核心原則（5 條以內，每次作業必遵守）
### 4. 禁止事項（容易犯錯的地方）

## 必含的 3 條核心原則

以下 3 條取代過去 `/understand`、`/new-feature` command 的功能，必須寫進每個專案的 CLAUDE.md：

1. 接到任務前，view `.claude/rules/` 下相關檔案以理解專案現況
2. 開新功能前，先讀 `todo-and-plans.md` 確認與計畫無衝突
3. 開始實作前，列出預計異動的檔案範圍給使用者確認

## 文件索引規則
- 只列實際存在的 rules 檔（依專案類型而定）
- 不列尚未建立的檔案

## 禁止放入 CLAUDE.md
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
- 架構：.claude/rules/architecture.md
- 技術棧：.claude/rules/tech-stack.md
- API 規範：.claude/rules/api-conventions.md
- 資料模型：.claude/rules/data-model.md
- 業務邏輯：.claude/rules/business-logic.md
- 測試策略：.claude/rules/testing-strategy.md
- 部署：.claude/rules/deployment.md
- TODO：.claude/rules/todo-and-plans.md

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
- 架構：.claude/rules/architecture.md
- 技術棧：.claude/rules/tech-stack.md
- Design Guide：.claude/rules/design-guide.md
- API（消費端）：.claude/rules/api-conventions.md
- 業務邏輯：.claude/rules/business-logic.md
- 測試策略：.claude/rules/testing-strategy.md
- 部署：.claude/rules/deployment.md
- TODO：.claude/rules/todo-and-plans.md

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
- 架構：.claude/rules/architecture.md
- 技術棧：.claude/rules/tech-stack.md
- API 規範：.claude/rules/api-conventions.md
- 資料模型：.claude/rules/data-model.md
- Design Guide：.claude/rules/design-guide.md
- 業務邏輯：.claude/rules/business-logic.md
- 測試策略：.claude/rules/testing-strategy.md
- 部署：.claude/rules/deployment.md
- TODO：.claude/rules/todo-and-plans.md

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
- 架構：.claude/rules/architecture.md
- 技術棧：.claude/rules/tech-stack.md
- Design Guide：.claude/rules/design-guide.md
- API（消費端）：.claude/rules/api-conventions.md
- 資料模型（local DB）：.claude/rules/data-model.md
- 業務邏輯：.claude/rules/business-logic.md
- 測試策略：.claude/rules/testing-strategy.md
- 部署（store 上架）：.claude/rules/deployment.md
- TODO：.claude/rules/todo-and-plans.md

## 核心原則
1. 接到任務前，view 相關 rules 檔
2. 開新功能前，先讀 todo-and-plans.md
3. 開始實作前，列出預計異動的檔案範圍
4. {專案特有規則}

## 禁止
- {容易犯的錯}
```
