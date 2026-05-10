---
name: design-brief
description: 收集 UI 設計需求、判定 mode、產出 tool-agnostic 設計 brief。可單獨執行，輸出可用於 Stitch、Claude Design、v0 或任何 AI 設計工具。
---

# Skill：Design Brief 收集

## 目的

與使用者互動收集 UI 設計需求，判定設計 mode，產出一份結構化的 design brief。輸出格式為 tool-agnostic，可直接用於 Stitch、Claude Design、v0 或任何 AI 設計工具，也可作為後續設計或實作 skill 的前置輸入。

## Modes

- `greenfield`：完全沒有既有 UI，或明確要求從零建立產品 UI。
- `feature-extension`：已有網站/app，要依目前風格新增功能、頁面或流程。
- `design-guide-refresh`：目標只是建立、整理或更新 `.agents/rules/design-guide.md`。

若需求同時符合多個 mode，選影響最大的，並在確認階段說明理由。

## 流程

### Step 1 — Mode 判定與意圖確認

依使用者描述判定 mode，輸出以下確認訊息後**等待使用者回覆**，不可跳過：

```text
## Mode 判定
- mode: greenfield | feature-extension | design-guide-refresh
- 判定理由:
- 我理解的目標:
- 預計設計範圍:
- 會讀取的既有資料:
- 需要你確認:
```

### Step 2 — 讀取專案文件

使用者確認 mode 後，並行讀取以下檔案（存在才讀，不存在直接略過）：

1. `.agents/rules/architecture.md` — 系統目的、模組
2. `.agents/rules/business-logic.md` — 主要功能與流程
3. `.agents/rules/design-guide.md` — 視覺與元件基準
4. `.agents/rules/tech-stack.md` — UI 套件與技術限制

四個讀取請求同時發起，全部返回後再進入 Step 3。

### Step 3 — 補充必要資訊

依 mode 檢查必要資訊是否齊全。**優先從 Step 2 讀到的文件取值，缺必要資訊才問使用者**，不可用大量假設填充。

**greenfield 必要資訊**
- 產品目的
- 目標使用者
- 平台（Web / mobile / both）
- 核心功能或主要工作流
- 主要畫面或資訊架構
- 視覺調性（極簡/商務/活潑/嚴肅 + 1–2 形容詞）
- 色彩偏好（主色 hex 或「無偏好」）
- scope：要產出哪些頁面、每頁主要操作

**feature-extension 必要資訊**
- 新功能要解決的問題
- 新功能入口位置
- 影響的頁面、流程或元件
- 主要使用者操作流程（≥ 3 步驟：觸發 → 操作 → 結果）
- 既有風格來源（design-guide、現有頁面、截圖、可跑的 localhost 或程式碼）← **硬性必要**
- 互動狀態需求（loading / empty / error 是否需要設計）
- 技術限制與不可違反的專案規則

**design-guide-refresh 必要資訊**
- 現有 UI/元件來源
- 需要刷新或補齊的設計規範範圍
- 專案 UI 套件與禁止事項

資訊收集完成後，輸出足夠性檢查：

```text
## 資訊足夠性檢查
- 已知資訊:
- 缺少的必要資訊:
- 可安全假設:
- 高風險假設:
- 是否可產出 brief: YES | NO
```

若 `是否可產出 brief` 為 `NO`，繼續詢問使用者補充，不可進入 Step 4。

### Step 4 — 產出 Brief

足夠性確認為 YES 後，同時輸出兩種格式：

**(a) 螢幕顯示**

````markdown
# Design Brief: {slug}
產生時間：{YYYY-MM-DD}
Mode：{greenfield | feature-extension | design-guide-refresh}

## 產品概覽
- 目的:
- 目標使用者:
- 平台:

## 功能範圍
- 主要功能 / 新增功能:
- 核心流程:
- 主要畫面:

## 視覺方向
- 調性:
- 色彩偏好:
- 互動狀態需求（loading / empty / error）:
- 黑名單（不要的元件或模式）:

## 技術限制
- UI 套件 / 框架:
- 不可違反的專案規則:

## 既有設計脈絡（feature-extension / design-guide-refresh）
- 既有 design-guide 摘要:
- 既有頁面 / 截圖 / 程式碼觀察:
- architecture / business-logic / tech-stack 摘要:

---

## 給 AI 設計工具的 Prompt
{完整自然語言 prompt，可直接貼入 Stitch、Claude Design、v0 等工具}
````

**(b) 寫入檔案**

路徑：`.agents/design/{slug}/brief.md`（衝突處理規則見注意區塊）。

## 注意

- design-brief 只負責收集與輸出，**不呼叫任何設計工具 MCP**。
- `feature-extension` 若缺既有風格來源，足夠性標記 NO，不可假設風格。
- 「給 AI 設計工具的 Prompt」使用中性格式，不預設任何特定工具的語法。
- slug 由使用者提供或從功能描述自動產生（kebab-case，英文或拼音）。
- **brief.md 不存在**（含目錄不存在）：直接寫入。
- **brief.md 已存在**：詢問使用者覆蓋或另存為 `brief-{YYYY-MM-DD}.md`。
