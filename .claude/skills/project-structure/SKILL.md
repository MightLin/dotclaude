# Skill：建立標準專案結構

## 標準 .claude/ 目錄配置
```
.claude/
├── rules/
│   ├── architecture.md
│   ├── tech-stack.md
│   ├── design-guide.md      ← 有 UI 規範才建
│   ├── api-conventions.md   ← 有 API 規範才建
│   └── todo-and-plans.md
└── commands/
    └── understand-project.md
```

## understand-project.md 固定內容
```markdown
依序 view 以下所有檔案，完整理解專案後再開始作業：
1. view .claude/rules/architecture.md
2. view .claude/rules/tech-stack.md
3. view .claude/rules/api-conventions.md（若存在）
4. view .claude/rules/design-guide.md（若存在）
5. view .claude/rules/todo-and-plans.md
```

## 原則
- 不確定是否需要某個 rule 檔時，先問使用者
- 每個 rule 檔開頭都加上最後更新日期
- rules/ 下不放流程步驟，只放「知識與規範」
