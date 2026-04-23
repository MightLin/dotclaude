# Skill：撰寫精簡的 CLAUDE.md

## 核心原則
CLAUDE.md 每次 session 都會被讀入，直接消耗 token。
應該「小而精準」，只放每次都需要的資訊。

## 必要內容（不超過 40 行）

### 1. 專案一句話說明
### 2. 文件索引（指向 .claude/rules/ 下的檔案）
### 3. 核心原則（5 條以內，每次作業都必須遵守的）
### 4. 禁止事項（容易犯錯的地方）

## 禁止放入 CLAUDE.md 的內容
- 完整的功能說明或流程描述（放 architecture.md）
- 技術棧細節（放 tech-stack.md）
- TODO 清單（放 todo-and-plans.md）
- 任何超過 2 行的說明

## 範例範本
```markdown
# ProjectName

{一句話說明專案目的}

## 文件索引
- 架構與模組：.claude/rules/architecture.md
- 技術棧：.claude/rules/tech-stack.md
- API 規範：.claude/rules/api-conventions.md
- Design Guide：.claude/rules/design-guide.md
- 業務邏輯：.claude/rules/business-logic.md
- TODO：.claude/rules/todo-and-plans.md

## 核心原則
1. 修改前先 view 相關檔案，不整份重寫
2. 新功能開發前執行 /project:understand
3. {專案特有規則 1}
4. {專案特有規則 2}

## 禁止
- {容易犯的錯 1}
- {容易犯的錯 2}
```
