---
name: design-brief
description: 收集 UI 設計需求、判定 mode、產出 tool-agnostic 設計 brief。可單獨執行，輸出可用於 Stitch、Claude Design、v0 或任何 AI 設計工具。
updated: 2026-05-16
version: 0.3.0
---

# Skill：Design Brief 收集

## 目的

與使用者互動收集 UI 設計需求，判定設計 mode，產出一份結構化、可直接餵給 AI 設計工具的 design brief。輸出為 tool-agnostic，可用於 Stitch、Claude Design、v0 等。

依 mode 不同產出不同模板，視覺規格要具體到 hex 與 px，避免抽象形容詞稀釋 prompt 訊號。

## Modes

- `greenfield`：完全沒有既有 UI，或明確要求從零建立產品 UI。
- `feature-extension`：已有網站/app，要依目前風格新增功能、頁面或流程。

若使用者需求是要建立或更新 `.agents/rules/design-guide.md`，請改用 `/design-guide` skill，不在本 skill 處理。

若需求同時符合多個 mode，選影響最大的，並在確認階段說明理由。

## 流程

### Step 1 — Mode 判定與意圖確認

依使用者描述判定 mode。**判定路徑分兩種**：

**(a) 描述足以判定 mode**：輸出以下確認訊息後**等待使用者回覆**，不可跳過：

```text
## Mode 判定
- mode: greenfield | feature-extension
- 判定理由:
- 我理解的目標:
- 預計設計範圍:
- 會讀取的既有資料:
- 需要你確認:
```

**(b) 描述不足以判定 mode**（例如使用者只說「幫我設計 UI」、「幫我做一個 app」、未提及既有專案狀態）：**必須呼叫 `AskUserQuestion`** 提供兩選項按鈕，禁止以散文要求使用者補充。兩選項說明如下：

- **greenfield** — 從零建立全新產品 UI。適用：新 side project、新模組、明確說「全部重做」、沒有既有 design-guide。
- **feature-extension** — 在現有 app/網站加新頁面或新功能。適用：既有 codebase 有 design-guide 或可觀察的既有頁面，新功能要融入既有視覺。

若使用者實際是要刷新 design-guide 本身，請告知改用 `/design-guide` skill。

使用者透過按鈕選定 mode 後，再輸出 (a) 的 Mode 判定確認訊息。

### Step 2 — 讀取專案文件

使用者確認 mode 後，並行讀取以下檔案（存在才讀，不存在直接略過）：

1. `.agents/rules/architecture.md` — 系統目的、模組
2. `.agents/rules/business-logic.md` — 主要功能與流程
3. `.agents/rules/design-guide.md` — 視覺與元件基準（**feature-extension 必須讀**）
4. `.agents/rules/tech-stack.md` — UI 套件與技術限制

四個讀取請求同時發起，全部返回後再進入 Step 3。

### Step 3 — 補充必要資訊

依 mode 檢查必要資訊是否齊全。**優先從 Step 2 讀到的文件取值，缺必要資訊才問使用者**，不可用大量假設填充。

**greenfield 必要資訊**
- 產品目的
- 目標使用者
- 平台（Web / mobile / both）
- 核心功能或主要工作流
- 主要畫面或資訊架構（**含每個畫面的視覺重點**）
- 視覺調性（極簡/商務/活潑/嚴肅 + 1–2 形容詞）
- **視覺風格參考**（≥1 個產品名 + 風格形容詞，例：Cal.com + Linear, 簡約現代）
- **色彩系統**（≥6 個 hex，需含 primary / surface / border / text-primary / text-secondary / error；若使用者無偏好，由 AI 依調性提案完整 hex 表後讓使用者確認）
- **元件尺寸基準**（spacing 單位、預設 radius、預設 row/chip 高度）
- **黑名單**（明確不要的元件或視覺模式）

**feature-extension 必要資訊**
- 新功能要解決的問題
- 新功能入口位置
- 影響的頁面、流程或元件
- 主要使用者操作流程（≥ 3 步驟：觸發 → 操作 → 結果）
- **既有風格來源**（design-guide、現有頁面、截圖、可跑的 localhost 或程式碼）← **硬性必要**
- **既有 design-guide token 摘要**（必須從 `.agents/rules/design-guide.md` 讀出 hex 與尺寸；讀不到才問使用者，禁止另外發明 hex）
- 新功能畫面與既有導航/IA 的銜接點
- 新增元件 vs 沿用元件清單
- 互動狀態需求（loading / empty / error 是否需要設計）
- 技術限制與不可違反的專案規則
- 黑名單（沿用既有規範 + 本次特別禁止項）

資訊收集完成後，輸出足夠性檢查（**僅對使用者顯示，不寫進最終 brief.md**）：

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

足夠性確認為 YES 後，依 mode 選擇對應模板，同時輸出螢幕顯示 + 寫入檔案。

**共通規則**：
- 強制 YAML frontmatter：`name / description / mode / date / slug` 五欄皆必填
- **不寫入**「資訊足夠性檢查」段（那只在對話階段顯示）
- 「給 AI 設計工具的 Prompt」段**雙語並陳**：先英文 fenced code block（給工具直接貼）、後附中文版（給人對照）

#### Template A：greenfield

````markdown
---
name: {人類可讀的設計名稱}
description: {一句話描述設計範圍}
mode: greenfield
date: {YYYY-MM-DD}
slug: {kebab-case}
---

# Design Brief: {slug}
產生時間：{YYYY-MM-DD}
Mode：greenfield

## 產品概覽
- **目的:**
- **目標使用者:**
- **平台:**

## 功能範圍
- **設計範圍:**
- **核心流程:** (列出步驟 1→2→3→...)
- **主要畫面:** (逐畫面，每個畫面附 1–2 句視覺重點)

## 視覺方向

### 風格定調：{產品名/概念}
參考感：{產品 A + 產品 B + 形容詞}

**核心概念**
- {3–5 條視覺核心原則}

### 色彩系統
| 角色 | 色票 | 用途 |
|------|------|------|
| Primary | `#XXXXXX` | ... |
| Surface | `#XXXXXX` | ... |
| Border | `#XXXXXX` | ... |
| Text-primary | `#XXXXXX` | ... |
| Text-secondary | `#XXXXXX` | ... |
| Error | `#XXXXXX` | ... |
| ...其他角色 | | |

### 字型與間距
- Font: ...
- Base size: ...
- Spacing 單位: ...
- 圓角: ...

### 元件規格
- **{元件名}：** {具體 px、色、行為}
- ...

## 技術限制
- **UI 套件:**
- **CSS / 樣式系統:**
- **不可使用:**

## 黑名單
- ...

---

## 給 AI 設計工具的 Prompt

### English (for Stitch / Claude Design / v0)

```
{完整英文 prompt，自包含；含 layout structure、visual style、typography、tone、輸出指示}
```

### 中文對照
{對應的中文版 prompt}
````

#### Template B：feature-extension

````markdown
---
name: {人類可讀的設計名稱}
description: {一句話描述新功能}
mode: feature-extension
date: {YYYY-MM-DD}
slug: {kebab-case}
extends-design-guide: true | false
---

# Design Brief: {slug}
產生時間：{YYYY-MM-DD}
Mode：feature-extension

## 既有設計脈絡
- **既有 design-guide 摘要（從 `.agents/rules/design-guide.md` 摘出）:**
  - 主色系: `#XXXXXX` / ...
  - 字型: ...
  - Spacing 單位: ...
  - 既有核心元件: ...
- **既有頁面 / 截圖 / 程式碼觀察:** (若有 localhost 或既有頁面，需明確指認)
- **architecture / business-logic / tech-stack 摘要:**

## 新功能範圍
- **要解決的問題:**
- **入口位置:**
- **影響的頁面、流程、元件:**
- **核心操作流程:** 觸發 → 操作 → 結果

## 新功能視覺
- **沿用既有元件:** {列表}
- **新增元件:** {列表 + 為何不能沿用既有}
- **新增畫面視覺重點:** (逐新增畫面)

## 互動狀態需求
- Loading / Empty / Error / ...

## 技術限制
- **UI 套件:**
- **不可違反的專案規則:**

## 黑名單
- ...

---

## 給 AI 設計工具的 Prompt

### English (for Stitch / Claude Design / v0)

```
{英文 prompt，必須附 "must match existing design system" context，明確引用既有 design-guide 的 token；著重描述新增畫面/元件}
```

### 中文對照
{對應中文版}
````

#### 寫入檔案

路徑：`.agents/design/{slug}/brief.md`（衝突處理規則見注意區塊）。

## 注意

- design-brief 只負責收集與輸出，**不呼叫任何設計工具 MCP**。
- Mode 判定不確定時**必須用 `AskUserQuestion`** 跳兩選項按鈕，禁止以散文要求使用者用文字補充 mode。
- 若使用者實際需求是刷新 design-guide 本身，請改用 `/design-guide` skill，不在本 skill 處理。
- 「資訊足夠性檢查」是對話階段工具，**不可寫進最終 brief.md**。
- frontmatter 5 欄位皆必填，不可省略；`feature-extension` 多 `extends-design-guide`。
- AI prompt 段**必須雙語並陳**，英文段需為自包含 fenced code block。
- `feature-extension` 若缺既有風格來源，足夠性標記 NO，不可假設風格；色票必須從 `.agents/rules/design-guide.md` 摘出，禁止另外發明 hex。
- 色彩系統若使用者無偏好，由 AI 依調性提案完整 hex 表後讓使用者確認，不可只寫「無偏好」帶過。
- slug 由使用者提供或從功能描述自動產生（kebab-case，英文或拼音）。
- **brief.md 不存在**（含目錄不存在）：直接寫入。
- **brief.md 已存在**：詢問使用者覆蓋或另存為 `brief-{YYYY-MM-DD}.md`。

## Changelog

### 0.3.0 — 2026-05-16
- mode 判定不確定時改用 `AskUserQuestion` 按鈕，禁止使用者手 key
- 拆出 greenfield / feature-extension 兩個獨立 brief 模板（Template A / B）
- frontmatter 改為硬性必填，feature-extension 多 `extends-design-guide`
- 「資訊足夠性檢查」只在對話顯示、不寫進 brief.md
- greenfield 必填欄加上 hex 色票表、元件 px 規格、視覺風格參考
- feature-extension 必填欄改為強制從既有 design-guide 摘錄 token，禁止另外發明 hex
- AI prompt 段改雙語並陳（英文 fenced block + 中文版）

### 0.2.0 — 2026-05-16
- 移除 `design-guide-refresh` mode，改由 `/design-guide` skill 自行釐清

### 0.1.0 — 2026-05-11
- 初始版本：design brief 收集、mode 判定、brief 輸出流程
