## 規劃中 / 草稿 Skills

### stitch-design（暫退出 skills/，放在 `_planning/stitch-design/`）

**現況**：流程架構已定（orchestration loop、generator、evaluator、rubric），但尚未穩定可用。

**待解決問題**：
- stitch-evaluator 讀取 rubric 的路徑依賴 dotclaude repo 結構，部署到其他專案後會失敗
- design-guide-refresh mode PASS 後的 handoff 政策未定義（是否允許更新 design-guide.md、需要哪種確認）
- 3 次 FAIL 後使用者選擇「重跑」的計數器重置語義不明確
- slug 命名在 design-brief 與 stitch-design 之間不一致
- handoff 目錄結構缺 design-guide-refresh 的專屬產物（draft-design-guide.md）

**重新啟用前需完成**：修正上述問題後，將 `_planning/stitch-design/` 移回 `skills/stitch-design/`，更新 version 與 updated，執行 sync-config。

---

# Stitch / Claude Design -> Codex / Claude Code

先朝最簡單的流程前進：

```text
你的想法
-> Stitch / Claude Design input
-> DESIGN.md + prototype
-> Codex / Claude Code input
-> repo 裡的實作
```

也可以理解成：

```text
Node 1: Stitch / Claude Design
-> Node 2: Codex / Claude Code
-> 真正的專案程式碼
```

## Node 1：Stitch / Claude Design

定位：把「模糊需求」變成「可實作的 UI 規格」。

### Input

```text
1. 產品目標
這個功能要解決什麼問題？

2. 使用者角色
誰會用？新手、管理員、交易員、一般用戶？

3. 頁面 / 功能範圍
要做 dashboard、設定頁、登入頁、表單、列表、詳情頁？

4. 內容資料
畫面上會有哪些欄位、數字、卡片、列表、操作按鈕？

5. 風格方向
專業、極簡、SaaS、金融感、developer tool、consumer app？

6. 裝置尺寸
先做 desktop、mobile，還是 responsive？

7. 參考素材
競品截圖、現有網站、品牌色、logo、舊畫面、手繪草圖。
```

### Output

```text
1. UI mockup / prototype
畫面長什麼樣子。

2. Layout 說明
頁面分區、資訊層級、主要操作在哪。

3. Component list
需要哪些元件：Navbar、Sidebar、Card、Table、Modal、Form、Tabs。

4. Interaction states
hover、active、loading、empty、error、success。

5. Responsive behavior
桌機、平板、手機怎麼變。

6. Visual rules
顏色、字體、間距、圓角、陰影、密度。

7. Handoff prompt / DESIGN.md
交給 Codex 或 Claude Code 實作的文字規格。
```

### Node 1 最小 prompt 範例

```text
請幫我設計一個 SaaS 後台 dashboard。

使用者是純工程背景的產品開發者。
目標是讓使用者快速看到系統狀態、任務進度、錯誤事件與近期活動。

頁面需要包含：
- 左側 sidebar
- 頂部搜尋與帳號區
- 4 個核心指標卡片
- 任務列表
- 錯誤事件表格
- 近期活動 timeline

風格：
- 專業、乾淨、工程工具感
- 不要行銷 landing page
- 資訊密度中高
- desktop first，但要支援 responsive

請輸出：
1. UI 設計方向
2. component list
3. desktop / mobile layout
4. loading / empty / error states
5. 可以交給 Codex 實作的 handoff prompt
```

## Node 2：Codex / Claude Code

定位：把「UI 規格」變成「repo 裡可跑、可維護的程式碼」。

### Input

```text
1. Handoff prompt / DESIGN.md
從 Stitch 或 Claude Design 來的設計規格。

2. Screenshot / prototype link
讓 AI 知道視覺結果。

3. 現有 repo context
技術棧、router、component structure、CSS 方案、設計規則。

4. 實作範圍
要新增哪個頁面？改哪個 component？接不接 API？

5. 資料結構
mock data、API response、欄位定義。

6. 驗收標準
跑得起來、responsive、狀態完整、符合截圖、無 lint/test error。

7. 限制條件
不要引入新 UI library、不要改 backend、不要重構無關檔案。
```

### Output

```text
1. 可執行的程式碼
頁面、元件、樣式、狀態邏輯。

2. Mock data 或 API integration
先用假資料，或直接串真 API。

3. Responsive implementation
desktop / mobile 都能用。

4. UI states
loading、empty、error、success。

5. 測試或驗證
lint、unit test、browser screenshot、manual QA。

6. 變更摘要
改了哪些檔案、怎麼驗證。

7. 後續 TODO
哪些可以之後再補。
```

### Node 2 最小 prompt 範例

```text
請根據以下 DESIGN.md 實作這個 dashboard 頁面。

限制：
- 使用現有專案技術棧與 component pattern
- 不要引入新的 UI library
- 先使用 mock data
- desktop first，但 mobile 不能壞
- 請實作 loading、empty、error state
- 完成後請跑 lint/test，並用瀏覽器檢查畫面

請先讀專案結構與相關檔案，再列出你會修改的檔案範圍。
接著實作、驗證，最後回報修改內容與測試結果。

DESIGN.md:
...
```

## 最小交接格式

可以先不要搞太複雜，只要讓 Node 1 固定產出這個格式：

```md
# DESIGN.md

## Goal
這個畫面要解決什麼問題。

## User
誰會使用。

## Screen
要做哪個頁面。

## Layout
頁面區塊與資訊層級。

## Components
需要哪些元件。

## Data
畫面需要哪些資料欄位。

## States
loading / empty / error / success。

## Responsive
desktop / mobile 如何排列。

## Visual Style
顏色、間距、密度、字體感覺、圓角。

## Implementation Notes
給 Codex / Claude Code 的實作注意事項。
```

重點放在 `DESIGN.md`。

它是這條流程的接頭。寫得好，後面不管是 Codex、Claude Code，甚至人類工程師接手，都會順很多。
