---
name: write-architecture-rules
description: Write or update concise project architecture rules. Use when initializing or maintaining `.agents/rules/architecture.md` with system overview, module responsibilities, data flow, and dependency boundary guidance for frontend, backend, fullstack, or mobile projects. For pure tech choices (libraries, versions) use `write-tech-stack-rules`; for API contract details use `write-api-conventions-rules`; for visual/component rules use `write-design-guide-rules`; for domain logic use `write-business-logic-rules`.
updated: 2026-05-27
version: 0.3.0
---

## Changelog

### 0.3.0 - 2026-05-27
- 新增 Source of Truth 原則，讓架構 rules 優先保存職責、資料流與邊界，而非完整檔案清單。

### 0.2.0 - 2026-05-27
- skill 改名為 `write-architecture-rules`，讓 skill 名稱描述撰寫/維護 rules 的動作，並保留產出檔名不變。

### 0.1.2 - 2026-05-25
- description 加入跨 rules skill 分流提示（write-tech-stack-rules / write-api-conventions-rules / write-design-guide-rules / write-business-logic-rules 的邊界）

### 0.1.1 - 2026-05-23
- 大小上限段新增超出時使用 rules-overflow skill 的提示。

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：撰寫架構 rules

## 目的
讓 Claude 快速理解系統的組成、各模組的職責邊界、資料流向。

## Source of Truth 原則

- rules 只保存 source 不容易快速推論的系統目的、模組職責、資料流與依賴邊界。
- 完整目錄樹、完整檔案清單、已集中在 source 的常數表或 registry，應改寫成摘要 + source pointer。
- 開發中專案可暫時記錄較厚的過渡架構，但需標示未來可收斂條件。

## 適用範圍
- 必要：所有專案類型（frontend / backend / fullstack / mobile）
- 不需要：單檔 script、experiment 倉

## 必要內容

### 1. 系統概述（3 句以內）
專案是什麼、服務誰、核心價值。

### 2. 模組清單
每個模組一行：
`- ModuleName：職責說明（相關目錄或命名空間）`

### 3. 資料流（文字或 ASCII）
主要業務流程的資料怎麼流動。依專案類型常見走向：
- 前端：使用者操作 → Component → Store/Service → HTTP Client → 後端 API
- 後端：Handler → Service → Repository → DB / 外部 API
- 全端：合併以上
- 手機：UI → ViewModel/Controller → Repository → Local DB / Remote API

### 4. 重要邊界規則
哪些模組不應直接互相依賴、哪些是共用核心。

## 禁止放入
- 完整檔案清單（看程式碼即可）
- 套件版本（屬 tech-stack.md）
- TODO 與計畫（屬 todo-and-plans.md）
- 任何超過 5 行的描述段落

## 大小上限
產出檔案不超過 60 行。
超出時請使用 rules-overflow skill 與使用者協作決定壓縮或分離。

## 範例
```markdown
# 系統架構
最後更新：YYYY-MM-DD

## 系統概述
{一句話說明系統目的與服務對象}

## 模組清單
- {ModuleA}：{職責}（{路徑}）
- {ModuleB}：{職責}（{路徑}）
- {Shared}：{共用核心}（{路徑}）

## 資料流
{使用者操作} → {Component} → {Service} → {API} → {DB / 外部}

## 邊界規則
- {ModuleA 不直接呼叫 ModuleB，透過 Shared}
- {外部 API 只允許在 {層名} 發起}
```
