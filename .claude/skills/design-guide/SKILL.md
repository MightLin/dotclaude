# Skill：撰寫 design-guide.md

## 目的
讓 Claude 產出的 UI 程式碼符合專案既有的視覺規範與元件使用慣例。

## 適用範圍
- 必要：frontend / fullstack / mobile
- 不需要：純後端、CLI、library

## 必要內容（依專案類型挑用）

### 共通段
- 使用的 UI 套件 / 設計系統名稱與版本
- 主色 / 警告色 / 字型階層（若有規範）
- 暗色模式策略
- i18n / RTL 處理（若有）

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
- 完整色票表（屬 design tokens 檔，不屬 rules）
- 元件 API 文件（屬套件文件）
- 截圖（rules 是純文字）

## 大小上限
產出檔案不超過 80 行。

## 範例

### Web
```markdown
# Design Guide
最後更新：YYYY-MM-DD

## UI 套件
- {套件}：{版本}

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
