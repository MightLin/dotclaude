---
name: update-rules
description: Apply targeted fixes to existing `.agents/rules/` files based on audit-rules findings. Use after running audit-rules to resolve Rule Quality issues (ambiguous, unverifiable, overlapping, or conflicting rules) and Template Compliance violations — without rewriting entire rule files. Coverage Gaps are out of scope; this skill redirects those to the appropriate write skill.
updated: 2026-05-23
version: 0.1.1
---

## Changelog

### 0.1.1 - 2026-05-23
- Step 1 補充「剛才跑過了」與「貼上」的差異處理說明
- Step 2 新鮮度改為半轉介（提示執行 write skill 或使用者手動提供內容）
- Step 3 新增新鮮度類型的提案格式，明確說明無法自動產生修補建議

### 0.1.0 - 2026-05-23
- 建立初始 skill 規範。

# Skill：update-rules — 針對 audit-rules 報告的手術式修補

## 目的

承接 `audit-rules` 產出的報告，對 `.agents/rules/` 中的 Rule Quality 問題與 Template Compliance 違規執行逐一確認的手術式修補，避免整份 rules 重寫帶來的資訊流失。

Coverage Gap（程式碼有模式但 rules 未涵蓋）**不在本 skill 範疇**；這類問題應交給對應的 write skill（如 `architecture`、`tech-stack`）。

## 適用場景

- 剛跑完 `audit-rules`，想立刻修補 findings
- 已有舊版 audit 報告，想補做修補工作
- 不需要整份重寫，只要針對具體問題做最小範圍修正

## Step 1：取得 audit-rules 報告

詢問使用者報告來源（擇一）：

```
請問 audit-rules 報告從哪裡取得？

1. 剛才在對話中已產出（說「剛才跑過了」即可，或直接貼上報告文字）
2. 請你現在執行 audit-rules（我會先中止，讓你跑完再回來）
```

- 若使用者說「剛才跑過了」：從本次對話 context 中擷取最近一次 `audit-rules` 的輸出，作為報告來源，不再要求使用者貼上。
- 若使用者貼上報告文字：直接解析貼上的內容。
- 若使用者回答 2：說明「請先執行 `/audit-rules`，完成後再執行 `/update-rules`」，然後結束本 skill。
- 若報告中**無任何 finding**（Coverage Gap / Rule Quality / Template Compliance 均為空）：回覆「audit-rules 未發現需修補的問題，無需執行 update-rules。」並結束。

## Step 2：分類 findings

讀取報告，將所有 finding 分為三類：

| 類型 | 本 skill 的處理方式 |
|---|---|
| **Coverage Gap** | 轉介：提示執行對應 write skill，本 skill 不處理 |
| **Rule Quality**（模糊 / 不可驗證 / 重疊 / 衝突） | ✅ 主要處理對象 |
| **Template Compliance — 行數超限**（🔴 >100%） | 轉介：提示執行 `rules-overflow` skill |
| **Template Compliance — 新鮮度**（🟠/🔴） | 半轉介：提示使用者手動更新 rules 內容，或執行對應 write skill |
| **Template Compliance — 其他**（冗餘片段 / 可補章節 / 失效引用） | ✅ 處理 |

整理後輸出本次的工作清單：

```
## update-rules 工作清單

將處理（{n} 個）：
- [RQ] {finding 標題}  →  `.agents/rules/{file}.md:{line}`
- [TC] {finding 標題}  →  `.agents/rules/{file}.md`
...

轉介（{m} 個，不處理）：
- [Gap] {找到的 Coverage Gap 標題}  → 建議執行 `{write-skill}` skill
- [Overflow] `.agents/rules/{file}.md` 行數超限 → 建議執行 `rules-overflow` skill
- [Freshness] `.agents/rules/{file}.md` 新鮮度 {🟠/🔴} → 需手動更新或執行 `{write-skill}` skill
```

詢問使用者確認後才進入 Step 3。若使用者要跳過某些 finding，標記為「略過」並從工作清單移除。

## Step 3：逐一提案修補

對工作清單中每個 finding，依序進行以下流程（**不一次列出所有提案**）：

### 3.1 呈現現況與提案

```
── Finding {序號}/{總數} ──────────────────────────
類型：{Rule Quality — 模糊 / 不可驗證 / 重疊 / 衝突 | Template Compliance — 冗餘片段 / 可補章節 / 新鮮度 / 失效引用}
位置：`.agents/rules/{file}.md`{:line（若有）}

現況：
  {引用原始 rules 文字，含行號}

修改原因：
  {說明這條 finding 具體指出的問題}

建議修改：
- {原文字}
+ {建議文字}
（或：建議在 {位置} 新增章節 `## {section}` 並填入以下內容：...）

執行 / 跳過 / 自訂修改？
```

**新鮮度類型的提案例外**：新鮮度 finding 表示 rules 內容可能與程式碼脫節，無法從 audit 報告自動產生修補建議。遇到此類型時，改以下格式呈現：

```
── Finding {序號}/{總數} ──────────────────────────
類型：Template Compliance — 新鮮度 {🟠/🔴}
位置：`.agents/rules/{file}.md`
現況：rules 最後更新後，對應程式碼目錄有 {C} 個新 commit，內容可能已脫節。

此 finding 需比對最新程式碼才能確認修補範圍，超出本 skill 的手術式修補能力。
建議：
  a. 執行 `{對應 write-skill}` skill 重新產生此 rules 檔的相關段落
  b. 或自行提供更新後的規則文字，本 skill 協助套用

選擇 a / b / 跳過？
```

若使用者選擇 b，進入 Step 3.2 的「自訂修改」流程，由使用者提供新文字。

### 3.2 處理使用者回應

- **執行**：記錄此修改為待套用，繼續下一個 finding
- **跳過**：標記略過，繼續下一個 finding
- **自訂修改**：請使用者提供新文字，確認後記錄，繼續下一個 finding

所有 finding 審核完畢後才進入 Step 4。

## Step 4：套用核准的修補

1. 並行讀取所有需修改的 rules 檔（若本次對話已讀過且未被修改，不重讀）
2. 對每個核准的修改，執行手術式替換：
   - **文字替換**：只動 finding 相關的段落，不改其他行
   - **新增章節**：插入對應 `##` 段落，不動既有章節
   - **移除冗餘片段**：只刪命中「禁止放入」清單的區段，不動相鄰內容
   - **修正失效引用**：替換 rules 中已不存在的路徑字串
3. 若某 rules 檔的 front matter 有 `updated` 欄位，更新為今天的日期（`YYYY-MM-DD`）

**不在使用者確認前寫入任何檔案。**

## Step 5：輸出摘要

```
## update-rules 完成

修補了 {n} 個 finding，跨 {m} 個 rules 檔：
{- `.agents/rules/{file}.md`：{finding 標題} ×{count}}

略過：{k} 個（使用者選擇略過）

轉介（未在本 skill 處理）：
- Coverage Gap ×{gap_n} → 建議執行對應 write skill（見 audit-rules 報告「建議後續」）
- 行數超限 ×{overflow_n} → 建議執行 `rules-overflow` skill
- 新鮮度 ×{freshness_n} → 建議執行對應 write skill 或手動更新
```

若所有 finding 均被略過或轉介，摘要仍需列出轉介清單，不可只說「無修改」。

## 行為限制

- 不在使用者逐一確認前修改任何 rules 檔
- 不刪除整個 rules 檔
- 不整份重寫 rules 檔；每次修改只動 finding 相關段落
- 不處理 Coverage Gap（交給 write skills）
- 不自動執行 `rules-overflow`；只提示轉介
- 修改 front matter 只更新 `updated` 欄位，不動其他欄位
- 不修改 `.agents/rules/` 以外的檔案
