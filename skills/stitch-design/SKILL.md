---
name: stitch-design
description: Generate, evaluate, and hand off UI designs with Stitch MCP. Use when Codex needs to design UI from scratch, extend an existing website or app with a new feature in the current visual style, refresh `.agents/rules/design-guide.md`, run Stitch design generation, evaluate generated UI designs with a mode-aware rubric, or prepare an implementation handoff after the user approves a Stitch design.
---

# Skill：Stitch UI 設計工作流

## 目的
依使用者需求與專案現況，透過 Stitch MCP 產生 UI 設計。支援從零建立 UI、依既有網站風格新增功能、刷新 design guide。流程必須先確認 mode 與資訊足夠性，Stitch 設計完成後只提供 Stitch 連結給使用者確認；只有使用者明確表示要實作該方案後，才下載/export 或整理 handoff。

## Modes

- `greenfield`：完全沒有既有 UI，或使用者明確要求從零建立產品 UI。
- `feature-extension`：已有網站/app，要依目前風格新增功能、頁面或流程。
- `design-guide-refresh`：目標只是建立、整理或更新 `.agents/rules/design-guide.md`。

若使用者需求同時符合多個 mode，先選影響最大的 mode，並在確認階段說明原因。

## 必要 Gate

### 1. Mode 判定後先確認意圖
在呼叫 Stitch 或 subagent 前，主執行緒必須先回覆並等待使用者確認：

```text
## Mode 判定
- mode: greenfield | feature-extension | design-guide-refresh
- 判定理由:
- 我理解的目標:
- 預計設計範圍:
- 會讀取/使用的既有資料:
- 需要你確認:
```

使用者確認後才進入下一步。不可在未確認 mode 的情況下呼叫 Stitch。

### 2. 收集與檢查資訊
依 mode 收集資料。先讀可取得的專案文件，缺必要資訊才問使用者。

共同優先讀取：
1. `.agents/rules/architecture.md`（若存在）→ 系統目的、模組
2. `.agents/rules/business-logic.md`（若存在）→ 主要功能與流程
3. `.agents/rules/design-guide.md`（若存在）→ 視覺與元件基準
4. `.agents/rules/tech-stack.md`（若存在）→ UI 套件與技術限制

`greenfield` 必要資訊：
- 產品目的
- 目標使用者
- 平台（Web / mobile / both）
- 核心功能或主要工作流
- 主要畫面或資訊架構

`feature-extension` 必要資訊：
- 新功能要解決的問題
- 新功能入口位置
- 影響的頁面、流程或元件
- 既有風格來源（design-guide、現有頁面、截圖、可跑的 localhost 或程式碼）
- 技術限制與不可違反的專案規則

`design-guide-refresh` 必要資訊：
- 現有 UI/元件來源
- 需要刷新或補齊的設計規範範圍
- 專案 UI 套件與禁止事項

呼叫 Stitch 前必須輸出資訊足夠性檢查：

```text
## 設計前檢查
- 已知資訊:
- 缺少的必要資訊:
- 可安全假設:
- 高風險假設:
- 是否可呼叫 Stitch: YES | NO
```

若 `是否可呼叫 Stitch` 為 `NO`，先問使用者補資訊，不可用大量假設生成。

### 3. Orchestration Loop（最多 3 次）

```
attempt = 1
feedback = none

while attempt <= 3:
    gen_output = Agent(subagent_type="stitch-generator",
                       prompt=mode + brief + existing context + (feedback if feedback else ""))

    if gen_output starts with "ERROR:":
        中止，回報錯誤給使用者
        break

    eval_output = Agent(subagent_type="stitch-evaluator",
                        prompt=mode + brief + gen_output + existing context)

    解析 eval_output 取得分數、PASS/FAIL、feedback
    記錄此輪結果到歷史

    if PASS:
        break
    else:
        feedback = eval_output 的 Feedback 區塊
        attempt += 1
```

主執行緒不要重新評分，信任 evaluator 結果。

### 4. 設計方案確認
Stitch 生成完成後，確認設計方案時只提供 Stitch 連結，不下載、不複製、不產生 implementation handoff：

```text
## 設計方案
- Stitch 連結:
- 評估結果: PASS | FAIL
- 需要你確認:
```

若 Stitch 沒有提供可查看的 project link，必須回報限制，並請使用者決定是否改用文字摘要作為確認方式。不可假造連結。

### 5. 實作確認後才 handoff
只有當使用者明確表示要實作此方案（例如「就照這個實作」、「下載來 handoff」、「開始做」）後，才下載/export Stitch 產物或依 Stitch 連結整理實作交接資料。

實作產物放在：

```text
docs/designs/<feature-or-project-name>/
```

handoff 格式：

```text
## Implementation Handoff
- 來源 Stitch 連結:
- 下載/export 路徑:
- 目標:
- 頁面/routes/components:
- Layout 與 responsive:
- 元件使用:
- 狀態與互動:
- Accessibility:
- 驗收標準:
- 不可違反的專案規則:
```

## 注意

- Stitch 設計確認階段以 project link 為唯一主要交付物。
- MCP 回傳的大型 payload 留在子 agent 內，不回吐主執行緒。
- 不假設 Stitch MCP 一定支援下載/export；不支援時，實作階段改由 Stitch 連結內容整理 markdown handoff。
- 未經使用者確認，不覆蓋 `.agents/rules/design-guide.md`，也不把設計下載到專案資料夾。
