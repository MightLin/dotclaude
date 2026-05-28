---
name: write-todo-and-plans-rules
description: Write or update project TODO and planning rules. Use when initializing or maintaining `.agents/rules/todo-and-plans.md` for in-progress work, planned items, considering items, known issues, or when deciding whether an issue tracker replaces local planning rules.
updated: 2026-05-29
version: 0.3.1
---

## Changelog

### 0.3.1 - 2026-05-29
- 釐清 TODO rules 的保留範圍：保留當前計畫與少量仍影響實作的問題，完成歷史遷移到 tracker/changelog/docs。

### 0.3.0 - 2026-05-27
- 新增 Source of Truth 原則，明確將大量完成歷史、release history 與長期 backlog 導向 tracker/changelog/docs。

### 0.2.0 - 2026-05-27
- skill 改名為 `write-todo-and-plans-rules`，讓 skill 名稱描述撰寫/維護 rules 的動作，並保留產出檔名不變。

### 0.1.1 - 2026-05-23
- 大小上限段新增超出時使用 rules-overflow skill 的提示。

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：撰寫 TODO 與計畫 rules

## 目的
讓 Claude 知道哪些功能已規劃但未實作，避免重複設計或與計畫衝突。

## Source of Truth 原則

- rules 只保存短期 local context：In Progress、近期 Planned、Considering、Open Questions、少量仍會影響實作的 Known Issues。
- 大量已完成歷史、release history、PR 記錄或長期 backlog 應遷移到 issue tracker、`CHANGELOG.md`、`docs/history.md` 或 release notes。
- 若專案已有 GitHub Issues / Linear / Jira / Trello / Notion 作為 tracker，本檔應改為 pointer 或不建立。

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

### 5. Open Questions（開放問題）
會影響下一步實作方向、但尚未決定的產品或技術問題。

## 格式規範
- 每項一行：`- [ ] 功能名稱：簡短說明`
- 完成改 `- [x]`
- 重要的加優先序：P0 / P1 / P2
- 完成項只短暫保留到下一次 rules 維護；不要累積成歷史紀錄

## 禁止放入
- 細部實作步驟（屬 plan 檔，不屬 rules）
- 大量 `[x]`、PR/date history、release history 或已完成且不再相關的項目（遷移到 tracker/changelog/docs 或直接刪掉）
- 重複的描述（一行一項，沒有子敘述）

## 大小上限
產出檔案不超過 80 行。超過代表 backlog 太多，建議遷移至 issue tracker。
其他情境超出時請使用 rules-overflow skill 與使用者協作決定壓縮或分離。

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

## Open Questions
- [ ] {問題}：{決策會影響的範圍}
```
