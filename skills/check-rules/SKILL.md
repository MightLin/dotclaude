---
name: check-rules
description: Check whether a specified code scope violates existing project rules in `.agents/rules/`. Use for code compliance review against current rules only; do not audit rule coverage, rule completeness, or rule quality.
updated: 2026-05-23
version: 0.5.0
---

## Changelog

### 0.5.0 - 2026-05-23
- 將 Rules Gap 與 rules 品質審查拆出至 `audit-rules` skill
- 保留原本 diff 意圖優先的掃描範圍流程

### 0.4.0 - 2026-05-22
- 加入 Rules Gap 區塊，偵測程式碼反覆模式但 rules 未涵蓋的情況

### 0.3.0 - 2026-05-14
- 報告開頭加入 GREEN/YELLOW/RED 整體裁定
- Finding 格式改為 surgical redline：Rule 行引用規則原文，「問題」→「現況」，「建議修正」→「建議」
- 摘要行加入整體裁定結果

### 0.2.0 - 2026-05-11
- PR/diff 意圖時提示使用者確認後改讀 git diff 而非完整檔案
- 違規嚴重程度改為 [High / Medium / Low] 分級
- 報告加入 Unchecked Areas 區塊
- 加入 evidence-backed 原則：只列能引用具體位置的違規

### 0.1.0 - 2026-05-11
- 建立初始 skill 規範。

# Skill：check-rules — 程式碼合規檢查

## 目的

對照 `.agents/rules/` 的規範，掃描使用者指定的程式碼範圍，找出違規之處並提出具體修正建議。
適合手動撰寫的程式碼初稿、PR 送審前、或指定範圍的規則合規檢查。

此 skill 只檢查程式碼是否違反既有 rules；不判斷 rules 是否完整、不審查 rules 品質、不建議補 rules。若要檢查 rules 覆蓋缺口或 rules 檔品質，改用 `audit-rules` skill。

## 適用範圍

- 必要：專案已有 `.agents/rules/` 目錄（否則建議先執行 `init-project`）
- 不需要：純文件變更、只改 rules 檔本身

## Step 1：列出可用規則

列出 `.agents/rules/` 下所有 `.md` 檔。若該目錄不存在，中止並提示：「尚未初始化規則，請先執行 init-project skill。」

讀取每個 rules 檔的第一行（標題）與第一個 `##` 區段標題作為說明，依序編號輸出：

```
目前可用的規則檔：

1. architecture.md      — {第一個 ## 區段標題}
2. tech-stack.md        — {第一個 ## 區段標題}
（依實際存在的檔案列出，不補上不存在的檔案）

請輸入要對照的規則編號（逗號分隔），或輸入「全部」：
```

## Step 2：讀取選定的規則

並行讀取使用者選定的所有 rules 檔。

從每個 rules 檔提取**可與程式碼對照**的規範項目，聚焦於：
- 命名慣例（函式、檔案、欄位、路由等）
- 禁止使用的套件、函式、模式
- 必要的結構或格式（如回應格式、錯誤處理方式）
- 模組邊界（A 不得直接呼叫 B）

略過純描述性的段落（如系統概述、背景說明），這些不構成可查核的程式碼規則。

## Step 3：確認掃描範圍

若使用者的請求帶有 PR / diff / change / 這個 commit 等意圖，先告知使用者：

```
偵測到你想檢查的是變動的部分，我將只讀取 git diff 的範圍。
若你希望檢查完整專案，請告知。
```

確認後讀取 `git diff`，只檢查變動的 paths 與直接受影響的規範。

否則，詢問使用者要掃描的範圍：

```
請輸入要檢查的檔案或目錄（預設：整個專案）：
範例：src/api/  或  src/api/user.ts, src/api/order.ts
直接按 Enter 掃描整個專案
```

掃描整個專案時，自動排除：
- `node_modules/`, `vendor/`, `.git/`, `dist/`, `build/`, `.next/`, `.nuxt/`
- 二進位檔、圖片、`.lock` 檔

## Step 4：掃描程式碼並比對規範

並行讀取指定範圍的程式碼檔案。

對照 Step 2 提取的規範項目逐一比對，記錄每個違規：

- **檔案路徑與行號**（必須能引用具體位置，無法確認位置者不列為 finding）
- **嚴重程度**：
  - `[High]`：可能造成 correctness、security、data 或 user-facing regression
  - `[Medium]`：有意義的架構、測試或 workflow 違規
  - `[Low]`：命名、style 或輕微 convention drift
- **Rule**：來自哪個 rules 檔，並直接引用該規則的原文（讓使用者不用自己去查）
- **現況**：程式碼目前是什麼（事實描述，不評判）
- **建議**：應該改成什麼（給出可直接套用的修改）

無法從現有檔案驗證的規範（例如 rule 要求某 artifact 存在但找不到），列入 Unchecked Areas，不列為違規。

## Step 5：輸出報告

整體裁定規則（在輸出前先計算）：
- 🟢 **GREEN**：無 High、無 Medium → 可直接合併/上線
- 🟡 **YELLOW**：無 High 但有 Medium → 需人工確認後再決定
- 🔴 **RED**：任何 High → 必須修正才能合併

```
## check-rules 報告
對照規則：{選定的 rules 檔清單}
掃描範圍：{目錄、檔案或 git diff}

### 整體裁定：🔴 RED / 🟡 YELLOW / 🟢 GREEN

### Findings

- [High] {簡短問題標題}
  Evidence: `path/to/file.ext:line`
  Rule: `.agents/rules/example.md` — "{引用的具體規則文字}"
  現況：{程式碼目前是什麼}
  建議：{應該改成什麼}

- [Medium] ...

- [Low] ...

### Unchecked Areas

- {無法從現有檔案驗證的 rule，並說明原因}
（若全數可驗證，此區塊標示「無」）

### 摘要

掃描 {n} 個檔案，發現 {high} High / {medium} Medium / {low} Low 共 {total} 個 finding，整體裁定：{RED/YELLOW/GREEN}。
{若無 finding：「符合所有選定規範，整體裁定：🟢 GREEN。」}
```

## 行為限制

- 只讀取、比對、報告；不直接修改程式碼（除非使用者後續明確要求）
- 只引用能以具體檔案路徑與行號佐證的違規；推斷或假設不列為 finding
- project-local rules 與一般 best practices 衝突時，優先遵守 project-local rules
- 不審查 rules 覆蓋率或品質；若使用者想檢查 rules 缺口或品質，建議執行 `audit-rules`
- 不重新生成 rules 檔；若使用者想更新 rules，建議執行對應的 write skill
