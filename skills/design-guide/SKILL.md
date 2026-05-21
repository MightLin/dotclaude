---
name: design-guide
description: Write or update UI design rules. Use when initializing or maintaining `.agents/rules/design-guide.md` for UI libraries, design systems, layout, spacing, responsive behavior, components, forms, accessibility, mobile navigation, gestures, and platform design conventions.
updated: 2026-05-16
version: 0.2.0
---

# Skill：撰寫 design-guide.md

## 目的
讓 Claude 產出的 UI 程式碼符合專案既有的視覺規範與元件使用慣例。

## 流程

### Step 0 — 資訊收集與釐清

並行讀取以下檔案（存在才讀，不存在直接略過）：

1. `.agents/rules/design-guide.md` — 現有設計規範
2. `.agents/rules/tech-stack.md` — UI 套件與技術限制
3. `.agents/rules/architecture.md` — 系統目的

讀取完成後，輸出資訊足夠性檢查：

```text
## 資訊足夠性檢查
- 已知資訊:
- 缺少的必要資訊:
- 可安全假設:
- 是否可直接執行: YES | NO
```

**必要資訊**（需從專案檔案取得或使用者提供其中一項）：
- 現有 UI 套件或設計系統
- 要新增、修改或刪除的規範範圍

若 `是否可直接執行` 為 `NO`，列出缺少的必要資訊並等待使用者補充，不可進入下一步。
若為 `YES`，直接執行，不額外等待確認。

---

## 適用範圍
- 必要：frontend / fullstack / mobile
- 不需要：純後端、CLI、library

## 必要內容（依專案類型挑用）

### 共通段
- 使用的 UI 套件 / 設計系統名稱與版本
- 主色 / 警告色 / 字型階層（若有規範）
- 暗色模式策略
- i18n / RTL 處理（若有）
- 互動狀態規範：loading / empty / error 三種至少各一句（例：loading 用 Skeleton；empty 顯示說明文字與 CTA；error 用 inline 錯誤訊息）

### Web 段（frontend / fullstack）
- 佈局：grid、spacing、container 寬度、響應式斷點
- 元件慣例：列表 / 表單 / 對話框 / 通知 各對應哪個元件
- 表單規範：驗證觸發時機、錯誤訊息位置
- 禁止事項：禁用的元件或寫法（例如禁用 inline style、原生表單元素）
- 無障礙要求（aria 等級）

### Mobile 段（手機 app）
- 觸控目標最小尺寸（iOS 44pt / Android 48dp）
- 主要導航模式（Tab Bar / Drawer / Stack）
- 平台差異規則（iOS 用 iOS 慣例、Android 用 Material；或統一風格）
- 手勢規則（滑動、長按、下拉刷新）
- 鍵盤處理（avoid keyboard、focus 流）
- 安全區域（notch / home indicator）

## 禁止放入
- 完整色票表（屬 design tokens 檔，不屬 rules）；允許最小 token 集（主色 / 警告色 / 成功色 / 文字主次 disabled）
- 元件 API 文件（屬套件文件）
- 截圖（rules 是純文字）

## 設計來源索引
若有 `.agents/design/<slug>/index.md`，在 design-guide.md 頂部以一行指向它：
`設計來源：.agents/design/<slug>/index.md`
每頁結構樹、跨頁元件清單等超出 80 行限制的細節，一律拆到 `.agents/design/<slug>/` 子目錄。

## 大小上限
產出檔案不超過 80 行。超出細節（每頁結構樹、跨頁元件清單）一律拆到 `.agents/design/<slug>/`。

## 範例

### Web
```markdown
# Design Guide
最後更新：YYYY-MM-DD

## UI 套件
- {套件}：{版本}

## Tokens
```yaml
primary:     "{hex}"
danger:      "{hex}"
success:     "{hex}"
text:        "{hex}"
text-muted:  "{hex}"
spacing:     [4, 8, 12, 16, 24, 32]
breakpoints: { sm: 640, md: 768, lg: 1024 }
```

## 佈局
- 主內容區最大寬度：{px}
- 間距：{spacing scale}
- 響應式斷點：{清單}

## 元件慣例
- 列表：{元件}
- 表單欄位：{元件}
- 對話框：{元件}
- 通知：{元件}

## 表單規範
- 驗證時機：{onBlur / onSubmit}
- 錯誤訊息：{位置}

## 禁止
- 不使用 inline style
- 不使用原生 HTML 表單元素
```

### Mobile
```markdown
# Design Guide
最後更新：YYYY-MM-DD

## 設計系統
- {系統}（Material 3 / Cupertino / 自家系統）

## 觸控
- 最小目標：{44pt iOS / 48dp Android}
- 主要手勢：{滑動清單刪除、下拉刷新、長按}

## 導航
- 主架構：{Tab Bar / Drawer}
- 二級頁：{Stack push}
- 模態：{何時用 sheet / 全螢幕 modal}

## 平台差異
- {iOS / Android 慣例選擇}
- {字型 / 圖示風格}

## 鍵盤與安全區
- 鍵盤遮擋：{KeyboardAvoidingView / IMECompat}
- 安全區：{遵守 SafeAreaView / WindowInsets}

## 禁止
- 不違反平台 HIG / Material 規範
- {專案特定禁忌}
```

## Changelog

### 0.2.0 — 2026-05-16
- 加入 Step 0：自動讀取專案檔案、不足才問使用者

### 0.1.0 — 2026-05-10
- 建立初始 skill 規範
