# Skill：撰寫 todo-and-plans.md

## 目的
讓 Claude 知道哪些功能已規劃但未實作，避免重複設計或與計畫衝突。

## 必要內容

### 1. In Progress（進行中）
目前正在開發的功能。

### 2. Planned（已規劃）
確定要做但尚未開始，按優先序排列。

### 3. Considering（討論中）
還不確定要不要做，或做法未定。

### 4. Known Issues（已知問題）
目前存在的 bug 或技術債。

## 格式規範
- 每項一行，格式：`- [ ] 功能名稱：簡短說明`
- 完成後改為：`- [x] 功能名稱`
- 重要的加上優先序標記（P0 / P1 / P2）

## 範例
```markdown
# TODO & Plans
最後更新：YYYY-MM-DD

## In Progress
- [ ] 功能名稱：說明

## Planned
- [ ] 功能名稱：說明（P1）
- [ ] 功能名稱：說明（P2）

## Considering
- [ ] 功能名稱：評估中，原因說明

## Known Issues
- [ ] 問題描述：暫時解法或影響範圍
```
