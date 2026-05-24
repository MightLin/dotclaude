---
name: architecture
description: Write or update concise project architecture rules. Use when initializing or maintaining `.agents/rules/architecture.md` with system overview, module responsibilities, data flow, and dependency boundary guidance for frontend, backend, fullstack, or mobile projects. For pure tech choices (libraries, versions) use `tech-stack`; for API contract details use `api-conventions`; for visual/component rules use `design-guide`; for domain logic use `business-logic`.
updated: 2026-05-25
version: 0.1.2
---

## Changelog

### 0.1.2 - 2026-05-25
- description 加入跨 rules skill 分流提示（tech-stack / api-conventions / design-guide / business-logic 的邊界）

### 0.1.1 - 2026-05-23
- 大小上限段新增超出時使用 rules-overflow skill 的提示。

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：撰寫 architecture.md

## 目的
讓 Claude 快速理解系統的組成、各模組的職責邊界、資料流向。

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
