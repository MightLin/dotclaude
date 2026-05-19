---
name: ShiftTable 全系統重設計
description: 診所員工班表管理系統全面視覺重設計，範圍為主班表頁（月曆 grid + 員工側欄）
mode: greenfield
date: 2026-05-11
slug: shifttable-redesign
---

# Design Brief: shifttable-redesign
產生時間：2026-05-11
Mode：greenfield

## 產品概覽
- **目的:** 診所員工班表管理系統，供診所管理者安排月班表、查詢請假、執行 AI 自動排班、同步 Google Calendar
- **目標使用者:** 診所管理者（每日操作，高頻使用，需快速掃讀大量班次資料）
- **平台:** Web（全螢幕桌面，無需 mobile 考量）

## 功能範圍
- **設計範圍:** 主班表頁（月曆 grid + 員工側欄）
- **核心流程:**
  1. 管理者開啟系統 → 看到當月班表 grid（員工 × 日期）
  2. 點擊空格 → 指派班次（早/午/晚）
  3. 拖曳班次格 → 調整順序
  4. 觸發 AI 排班 → 對話框顯示進度 → 填入結果
  5. 查看右側違規清單 → 修正衝突
- **主要畫面:** 主班表頁（完整單頁應用，無需設計其他頁面）
- **班次類型:** 早班、午班、晚班（三種，日班可能混排）

## 視覺方向（建議）

### 風格定調：現代醫療排程（Modern Clinical Scheduler）
參考感：Cal.com + Linear + 簡約醫療 EHR

**核心概念**
- 主介面高度留白，讓密集的班次格數據清晰可讀
- 深色 Sidebar 形成視覺錨點，與白色主內容形成對比
- 班次類型以明確色塊區分，一眼辨識
- 操作按鈕低調但精準，不搶奪視線

### 色彩系統
| 角色 | 色票 | 用途 |
|------|------|------|
| Primary | `#4F46E5` Indigo-600 | 主要按鈕、選中狀態、月份導航 |
| Surface | `#FFFFFF` | 主內容背景 |
| Surface-2 | `#F8FAFC` | Toolbar、對話框背景 |
| Sidebar | `#1E1B4B` Indigo-950 | 員工側欄背景 |
| Border | `#E2E8F0` | 格線、分隔線 |
| Text-primary | `#0F172A` | 主要文字 |
| Text-secondary | `#64748B` | 次要標籤、日期 |
| 早班 | `#0EA5E9` Sky-500 | 早班 chip 背景 |
| 午班 | `#F59E0B` Amber-400 | 午班 chip 背景 |
| 晚班 | `#8B5CF6` Violet-500 | 晚班 chip 背景 |
| Error | `#EF4444` | 違規標記 |
| Warning | `#F97316` | 警告標記 |

### 字型與間距
- Font: System UI / `Inter`（若已安裝），16px base
- 班次格：緊湊但有 4px padding，chip 高度 20px
- 員工行高：48px
- 圓角：`4px` 格子、`6px` 按鈕/chip、`12px` 對話框

### 元件規格
- **Sidebar（員工列表）：** 深靛藍底（`#1E1B4B`），員工名稱白字，顏色指示圓點 12px，hover 時 `#312E81`
- **月曆 header row（日期列）：** 淺灰底（`#F1F5F9`），週六灰字，今日 Indigo 底色
- **班次 chip：** 圓角 4px，班次色底 + 白字，字號 12px，高 20px，可 drag 時顯示 grab cursor
- **Toolbar：** 白底，shadow-sm 底線，左側月份導航（`< 2026年5月 >`），右側動作按鈕群（AI排班 / 請假 / 同步）
- **違規清單 panel：** 右側抽屜或底部 panel，紅/橙 badge 標記嚴重度

## 技術限制
- **UI 套件:** PrimeNG 19+（Aura theme，強制 Light mode）
- **CSS:** Tailwind CSS 3.x（preflight: false）
- **不可使用:** PrimeFlex class、Angular Material、`pButton` directive（新按鈕用 `p-button` 元件）
- **例外:** `shift-cell` 中有 `cdkDrag` 的按鈕保留 `pButton` directive

## 既有設計脈絡
- **既有 design-guide 摘要:** PrimeNG Aura / Light mode / Tailwind for layout / 無 PrimeFlex
- **既有 tech-stack:** Angular 20 Standalone + Angular Signals，Drag-Drop via CDK
- **重設計意圖:** 完全推翻現有視覺風格，建立新設計語言；技術實作層（元件選用）維持不變

---

## 給 AI 設計工具的 Prompt

```
Design a web-based clinic employee shift scheduling application (desktop only, full-screen). 
The primary view is a monthly schedule grid showing employees × dates.

**Layout Structure:**
- Top toolbar: white background, subtle bottom shadow, left side has month navigation (< May 2026 >), right side has action buttons (AI Schedule, Leave Management, Sync Calendar)
- Left sidebar (fixed ~200px width): deep indigo background (#1E1B4B), white employee names with 12px color indicator dots
- Main area: white background monthly grid — rows = employees, columns = dates (Mon–Sat, Sunday is closed)
- Right panel (collapsible): rule violation list with red/orange severity badges

**Visual Style:**
- Clean, high-density data interface inspired by Cal.com and Linear
- Primary color: Indigo #4F46E5 for interactive elements and selected states
- Three shift types shown as color chips inside grid cells:
  - Early shift: Sky blue (#0EA5E9) chip
  - Afternoon shift: Amber (#F59E0B) chip  
  - Evening shift: Violet (#8B5CF6) chip
- Grid cells: white background, light gray border (#E2E8F0), 4px padding, shift chips are 20px tall with 4px border radius
- Subtle hover state on grid cells (light indigo tint)
- Today's date column has indigo header highlight

**Typography:**
- Font: Inter or system UI
- Base size: 14px for grid content, 12px for shift chips
- Employee names: 14px white on dark sidebar
- Date headers: 13px, gray for regular days, indigo for today

**Tone:** Professional, calm, medical-adjacent. Clean and functional without being sterile. 
Not playful — this is a daily operational tool for clinic managers.

Produce: Full-page monthly schedule view in desktop browser (1440px wide).
```
