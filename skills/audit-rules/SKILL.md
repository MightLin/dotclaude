---
name: audit-rules
description: Audit project rules for coverage gaps and rule quality. Use when reviewing whether `.agents/rules/` misses repeated code patterns, contains vague or unverifiable rules, overlaps across rule files, or needs maintenance suggestions.
updated: 2026-05-23
version: 0.1.0
---

## Changelog

### 0.1.0 - 2026-05-23
- 建立 rules 覆蓋缺口與 rules 品質審查 skill

# Skill：audit-rules — 規則覆蓋與品質審查

## 目的

審查 `.agents/rules/` 是否足以描述專案中穩定反覆出現的程式碼模式，並檢查 rules 檔本身是否清楚、可驗證、無重疊或衝突。

此 skill 只產出審查報告與維護建議；不直接修改 rules。若要檢查程式碼是否違反既有 rules，改用 `check-rules` skill。

## 適用範圍

- 必要：專案已有 `.agents/rules/` 目錄（否則建議先執行 `init-project`）
- 適合：定期 rules 維護、專案導入後校準、發現 rules 與實作脫節時
- 不適合：PR 合規檢查、單純找程式碼違規
- 掃描範圍：永遠對整個專案全域掃描，不支援 diff 模式。若要在 diff 範圍內做合規檢查，改用 `check-rules`。

## Step 1：讀取 rules 現況

列出 `.agents/rules/` 下所有 `.md` 檔。若該目錄不存在，中止並提示：「尚未初始化規則，請先執行 init-project skill。」

並行讀取所有 rules 檔，整理每個檔案的主題、可查核規則、描述性段落與可能的維護問題。

## Step 2：確認審查範圍

詢問使用者要審查的範圍：

```
請選擇 audit-rules 的審查範圍：

1. Coverage Gap + Rule Quality（預設）
2. 只檢查 Coverage Gap
3. 只檢查 Rule Quality
```

若使用者未指定，使用 Coverage Gap + Rule Quality。

## Step 3：Coverage Gap 審查

掃描專案程式碼，找出反覆出現但 `.agents/rules/` 未涵蓋的模式。

Coverage Gap 審查永遠以整個專案為範圍（非 diff 範圍），因為需要跨 ≥3 個檔案確認反覆模式是否形成專案級共識。

掃描整個專案時，自動排除：
- `node_modules/`, `vendor/`, `.git/`, `dist/`, `build/`, `.next/`, `.nuxt/`
- 二進位檔、圖片、`.lock` 檔

Coverage Gap 必須同時符合：

1. 同類模式出現在至少 3 個檔案
2. `.agents/rules/` 完全未提及該模式
3. 能明確歸屬到某個既有 rule 檔主題，或能建議新增哪類 rule 檔

不要把一般 best practices 當成缺口。只有 repo 程式碼已經存在穩定反覆模式，且 rules 完全未描述時，才列為 gap。

## Step 4：Rule Quality 審查

Rule Quality 審查只讀 `.agents/rules/` 檔本身，不掃程式碼。

審查 rules 檔本身，記錄具體問題：

- 模糊：規則無法指導實作，例如「保持乾淨」但沒有可觀察標準
- 不可驗證：無法從程式碼、設定或流程檢查是否遵守
- 重疊：多個 rules 檔描述同一規範，且可能造成維護分歧
- 衝突：不同 rules 對同一行為提出互斥要求
- 過長：內容超出該 rule skill 的大小上限或混入太多背景敘述
- 放錯檔案：內容應屬於另一個 rules 主題，例如 API response 格式放在 `architecture.md`

Rule Quality finding 必須引用具體 rules 檔路徑與行號；不需要引用程式碼位置。

## Step 5：輸出報告

```
## audit-rules 報告
審查範圍：{Coverage Gap + Rule Quality / Coverage Gap / Rule Quality}
Rules 檔：{實際讀取的 rules 檔清單}

### Coverage Gaps

- {反覆模式標題}
  Evidence: `{path1}`、`{path2}`、`{path3}`（至少 3 個檔案）
  現況：{程式碼中反覆出現的模式}
  缺口：`.agents/rules/` 未涵蓋 {主題}
  建議：執行 `{對應 write skill 名稱}` skill 更新 `.agents/rules/{file}.md`

（若無，標示「無」）

### Rule Quality Findings

- {品質問題標題}
  Evidence: `.agents/rules/{file}.md:{line}`
  類型：{模糊 / 不可驗證 / 重疊 / 衝突 / 過長 / 放錯檔案}
  現況：{rule 目前怎麼寫}
  建議：{應如何調整，並指向對應 write skill}

（若無，標示「無」）

### 建議後續

- {依 findings 建議下一步，例如執行 architecture / tech-stack / design-guide skill}
（若無，標示「無」）

### 摘要

發現 {gap_count} 個 coverage gap、{quality_count} 個 rule quality finding。
```

## 行為限制

- 只讀取、審查、報告；不直接修改 rules
- 不檢查程式碼是否違反既有 rules；這是 `check-rules` 的職責
- Coverage Gap 必須有至少 3 個檔案的反覆模式作為證據
- Rule Quality finding 必須引用具體 rules 檔路徑與行號
- 若建議更新 rules，指向對應 write skill，例如 `architecture`、`tech-stack`、`design-guide`
- project-local rules 與一般 best practices 衝突時，優先尊重 project-local rules
