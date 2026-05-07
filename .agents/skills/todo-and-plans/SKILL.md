---
name: todo-and-plans
description: Write or update project TODO and planning rules. Use when initializing or maintaining `.agents/rules/todo-and-plans.md` for in-progress work, planned items, considering items, known issues, or when deciding whether an issue tracker replaces local planning rules.
---

# Skill：撰寫 todo-and-plans.md

## 目的
讓 Claude 知道哪些功能已規劃但未實作，避免重複設計或與計畫衝突。

## 適用範圍
- 必要：所有有持續開發的專案
- 不需要：以 issue tracker（GitHub Issues / Linear / Jira）為唯一事實來源的專案。此時改在入口檔留 reference 指向 tracker。

## 必要內容

### 1. In Progress（進行中）
目前正在開發的功能。

### 2. Planned（已規劃）
確定要做但尚未開始，按優先序排列。

### 3. Considering（討論中）
還不確定要不要做或做法未定。

### 4. Known Issues（已知問題）
目前存在的 bug 或技術債。

## 格式規範
- 每項一行：`- [ ] 功能名稱：簡短說明`
- 完成改 `- [x]`
- 重要的加優先序：P0 / P1 / P2

## 禁止放入
- 細部實作步驟（屬 plan 檔，不屬 rules）
- 已完成且不再相關的項目（直接刪掉）
- 重複的描述（一行一項，沒有子敘述）

## 大小上限
產出檔案不超過 80 行。超過代表 backlog 太多，建議遷移至 issue tracker。

## 範例
```markdown
# TODO & Plans
最後更新：YYYY-MM-DD

## In Progress
- [ ] {功能}：{說明}

## Planned
- [ ] {功能}：{說明}（P1）
- [ ] {功能}：{說明}（P2）

## Considering
- [ ] {功能}：{評估中原因}

## Known Issues
- [ ] {問題}：{影響範圍 / 暫時解法}
```
