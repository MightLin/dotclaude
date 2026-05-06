# Skill：建立標準專案結構

## 標準 .claude/ 目錄配置

依專案類型條件式建立。`commands/` 預設不建（行為內化在 CLAUDE.md），有需要再建。

```
.claude/
└── rules/
    ├── architecture.md          ← 必建
    ├── tech-stack.md            ← 必建
    ├── business-logic.md        ← 有領域邏輯才建
    ├── todo-and-plans.md        ← 必建（除非用 issue tracker）
    ├── testing-strategy.md      ← 必建
    ├── deployment.md            ← 必建
    ├── api-conventions.md       ← 後端 / 全端必建；前端 / 手機若有自家規範可建
    ├── design-guide.md          ← 前端 / 全端 / 手機才建
    └── data-model.md            ← 後端 / 全端必建；手機有 local DB 才建
```

## 對應 skill 文件

| rules 檔 | skill |
|---|---|
| architecture.md | architecture |
| tech-stack.md | tech-stack |
| business-logic.md | business-logic |
| todo-and-plans.md | todo-and-plans |
| testing-strategy.md | testing-strategy |
| deployment.md | deployment |
| api-conventions.md | api-conventions |
| design-guide.md | design-guide |
| data-model.md | data-model |

## 原則
- 不確定是否需要某個 rule 檔時，先問使用者
- 每個 rule 檔開頭加最後更新日期
- rules/ 下只放「知識與規範」，不放流程步驟
- 建立前先依專案類型過濾，避免生空殼檔污染 context
