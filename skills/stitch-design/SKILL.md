---
name: stitch-design
description: Generate, evaluate, and hand off UI designs with Stitch MCP. Use when Codex needs to design UI from scratch, extend an existing website or app with a new feature in the current visual style, refresh `.agents/rules/design-guide.md`, run Stitch design generation, evaluate generated UI designs with a mode-aware rubric, or prepare an implementation handoff after the user approves a Stitch design.
updated: 2026-05-10
version: 0.1.0
---

## Changelog

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：Stitch UI 設計工作流

## 目的
依使用者需求與專案現況，透過 Stitch MCP 產生 UI 設計。支援從零建立 UI、依既有網站風格新增功能、刷新 design guide。流程必須先確認 mode 與資訊足夠性，Stitch 設計完成後只提供 Stitch 連結給使用者確認；只有使用者明確表示要實作該方案後，才下載/export 或整理 handoff。

## Modes

- `greenfield`：完全沒有既有 UI，或使用者明確要求從零建立產品 UI。
- `feature-extension`：已有網站/app，要依目前風格新增功能、頁面或流程。
- `design-guide-refresh`：目標只是建立、整理或更新 `.agents/rules/design-guide.md`。

若使用者需求同時符合多個 mode，先選影響最大的 mode，並在確認階段說明原因。

## 必要 Gate

### 1. 取得 Design Brief

執行 `/design-brief` 收集設計需求，取得：
- Mode（greenfield / feature-extension / design-guide-refresh）
- 結構化 Brief（產品概覽、功能範圍、視覺方向、技術限制、既有設計脈絡）
- 給 AI 設計工具的 Prompt
- 足夠性檢查結果（YES / NO）

若使用者已事先執行過 `/design-brief` 並提供 `.agents/design/{slug}/brief.md`，直接讀取該檔案，不重複收集。

若足夠性為 NO，不得繼續，等待 brief 補齊後再進行。

### 2. Orchestration Loop（最多 3 次）

```
attempt = 1
feedback = none

while attempt <= 3:
    # brief / mode / existing context 來自 design-brief 的輸出
    gen_output = Agent(subagent_type="stitch-generator",
                       prompt=mode + brief + existing_context + (feedback if feedback else ""))

    if gen_output starts with "ERROR:":
        中止，回報錯誤給使用者
        break

    eval_output = Agent(subagent_type="stitch-evaluator",
                        prompt=mode + brief + gen_output + existing_context)

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
- 評估結果: PASS | FAIL（附各維度分數）
- 嘗試次數:
- 需要你確認:
```

若 3 次全 FAIL，呈現所有輪次分數歷史，讓使用者選擇：接受最後一版 / 重跑 / 中止。

若 Stitch 沒有提供可查看的 project link，必須回報限制，並請使用者決定是否改用文字摘要作為確認方式。不可假造連結。

### 5. 實作確認後才 handoff
只有當使用者明確表示要實作此方案（例如「就照這個實作」、「下載來 handoff」、「開始做」）後，才下載/export Stitch 產物或依 Stitch 連結整理實作交接資料。

**若 3 次全 FAIL 且使用者接受最後一版**：只允許建立 `.agents/design/<slug>/`（標記「未通過評分」），禁止觸碰 `.agents/rules/design-guide.md`。

實作產物放在：

```text
.agents/design/<feature-or-project-name>/
  index.md            # 頁面清單 + Stitch 連結 + 設計摘要
  spec.md             # implementation spec
  pages/
    <page-name>.md    # 每頁的結構樹、區塊、元件、互動狀態
  raw/                # Stitch 下載的 HTML / 截圖（若 MCP 支援；不支援就空）
```

handoff 格式（`spec.md`）：

```text
# Implementation Spec: {feature-name}
產生時間：{YYYY-MM-DD}
mode：{greenfield | feature-extension | design-guide-refresh}
Stitch 來源：{逐頁 URL}

## 頁面 / 元件清單
## 佈局
## 視覺 tokens
## 互動流程
## 文案
## API / 資料需求
## 不做的事
```

## 注意

- Stitch 設計確認階段以 project link 為唯一主要交付物。
- MCP 回傳的大型 payload 留在子 agent 內，不回吐主執行緒。
- 不假設 Stitch MCP 一定支援下載/export；不支援時，實作階段改由 Stitch 連結內容整理 markdown handoff。
- 未經使用者確認，不覆蓋 `.agents/rules/design-guide.md`，也不把設計下載到專案資料夾。
- `feature-extension` 模式下禁止覆寫既有 design-guide.md；只允許新增段落或用 diff 呈現建議變更供使用者決定。
