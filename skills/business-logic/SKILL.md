---
name: business-logic
description: Write or update business/domain logic rules. Use when initializing or maintaining `.agents/rules/business-logic.md` for domain terminology, state flows, business constraints, permissions, workflows, billing, calculations, or other non-obvious product rules.
updated: 2026-05-10
version: 0.1.0
---

## Changelog

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：撰寫 business-logic.md

## 目的
讓 Claude 理解專案的業務領域知識，避免產出技術上正確但業務上錯誤的程式碼。

## 適用範圍
- 必要：有領域邏輯的專案（B2B、ERP、流程系統、計費、權限等）
- 不需要：純工具、純技術 demo、無領域語境的 library

## 必要內容

### 1. 名詞定義
專案特有的領域術語，一詞一義。只列「程式碼名稱無法直接推斷」的詞。

### 2. 核心業務流程
重要流程的狀態流轉：
`狀態A → 觸發條件 → 狀態B`

### 3. 業務規則與限制
明確的判斷條件：
`- 當 {條件} 時，{允許/禁止} {操作}`

### 4. 特殊計算邏輯
非直覺的計算方式或公式，附原因。

## 禁止放入
- 從 schema 或型別可推斷的欄位說明
- UI 文字、錯誤訊息文案
- 已實作完成、無外部知識的演算法（看程式碼即可）

## 大小上限
產出檔案不超過 100 行。超過代表領域過大，建議拆成多個檔（依子領域）。
其他情境超出時請使用 rules-overflow skill 與使用者協作決定壓縮或分離。

## 範例
```markdown
# Business Logic
最後更新：YYYY-MM-DD

## 名詞定義
- {術語}：{定義}

## 核心業務流程
{物件}狀態：草稿 → 送審 → 核准 → 進行中 → 結案
- 草稿：{誰可編輯}
- 送審：{鎖定條件}
- 核准後：{自動觸發的 side effect}

## 業務規則
- 當 {條件} 時，禁止 {操作}
- 當 {條件} 達 {閾值}，自動 {動作}
- {唯一性限制}

## 特殊計算
- {名稱}：{公式}（{原因}）
```
