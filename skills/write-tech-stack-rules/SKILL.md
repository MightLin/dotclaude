---
name: write-tech-stack-rules
description: Write or update technology stack rules. Use when initializing or maintaining `.agents/rules/tech-stack.md` for languages, versions, package managers, frameworks, UI libraries, state management, databases, infrastructure, external services, or banned alternatives.
updated: 2026-05-29
version: 0.3.1
---

## Changelog

### 0.3.1 - 2026-05-29
- 釐清 dependency version 的 source of truth：精確版本以 manifest / lockfile 為準，rules 只保存選型與 runtime 決策。

### 0.3.0 - 2026-05-27
- 新增 Source of Truth 原則，讓 tech-stack rules 優先保存技術決策與禁用替代方案。

### 0.2.0 - 2026-05-27
- skill 改名為 `write-tech-stack-rules`，讓 skill 名稱描述撰寫/維護 rules 的動作，並保留產出檔名不變。

### 0.1.1 - 2026-05-23
- 大小上限段新增超出時使用 rules-overflow skill 的提示。

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：撰寫技術棧 rules

## 目的
讓 Claude 知道用什麼技術、版本、慣用套件，避免給出不符合現況的建議。

## Source of Truth 原則

- rules 保存技術選型、主要 runtime 約束、package manager、禁止替代方案、跨工具相容性決策，以及不容易從 config 看出的原因。
- 精確 dependency version、patch/minor version、lockfile 狀態或套件 API 用法應指向 `package.json`、`pubspec.yaml`、`go.mod`、`requirements.txt`、lockfile 等 source，不複製進 rule。
- 可以保留「Node.js 22 runtime」「Flutter stable」「ESM / nodenext」這類 runtime / tooling decision；若只是套件版本，寫「以 manifest / lockfile 為準」。
- 開發中專案可暫存過渡選型；穩定後只保留主要 runtime/framework、package manager 與禁用項。

## 適用範圍
- 必要：所有專案類型
- 撰寫時依專案類型挑相關段落，不需要的整段省略

## 必要內容（依專案類型挑用）

### 共通段（所有類型）
- 語言 / runtime 約束（只保留主要版本或部署決策；精確 dependency version 指向 manifest）
- 套件管理工具
- 不使用的替代方案（避免 Claude 建議錯誤的套件）

### 後端段（backend / fullstack）
- 框架 / runtime 決策（精確套件版本以 manifest 為準）
- ORM 與資料庫
- HTTP 客戶端 / RPC client
- 背景工作機制（cron / queue / scheduler）
- 認證機制（JWT / Session / SSO）

### 前端段（frontend / fullstack）
- 框架 / rendering mode（React / Vue / Angular / Svelte / ...；精確套件版本以 manifest 為準）
- UI 套件
- 狀態管理方式
- HTTP 客戶端
- 路由與 SSR / SSG 設定
- 建置工具（Vite / Webpack / Turbopack / ...）

### 行動端段（mobile）
- 平台：iOS / Android / 跨平台
- 框架：Native（Swift/Kotlin）/ React Native / Flutter / Expo / KMP
- 狀態管理（Redux / Riverpod / Compose State / SwiftUI State）
- Local DB（SQLite / Realm / Room / Core Data）
- 推播 / 深層連結 / 離線同步策略

### 基礎設施段（所有類型）
- 部署環境
- CI/CD 工具
- 監控 / Logging
- 重要外部服務（金流、Email、儲存）

## 禁止放入
- 完整的 dependency 清單（看 lockfile 即可）
- 精確 dependency version 或 patch/minor 版本清單（看 manifest / lockfile 即可）
- 套件用法 / API 範例（屬使用文件，非規範）
- 已淘汰的舊套件（除非為了警告 Claude 不要用）

## 大小上限
產出檔案不超過 80 行。
超出時請使用 rules-overflow skill 與使用者協作決定壓縮或分離。

## 範例（依類型挑用）

### 純後端
```markdown
# 技術棧
最後更新：YYYY-MM-DD

## 共通
- Runtime：{主要版本或部署約束；精確 dependency version 以 manifest / lockfile 為準}
- 套件管理：{工具}

## 後端
- 框架：{框架 / major 或 runtime decision；精確版本看 manifest}
- ORM：{ORM} + {DB}
- HTTP 客戶端：{套件}（不使用 {替代方案}）
- 背景工作：{機制}
- 認證：{方式}

## 基礎設施
- 部署：{平台}
- CI/CD：{工具}
- 監控：{工具}
```

### 純前端
```markdown
# 技術棧
最後更新：YYYY-MM-DD

## 共通
- Runtime：{主要 runtime / 語言約束；精確 dependency version 以 manifest / lockfile 為準}
- 套件管理：{npm / pnpm / yarn}

## 前端
- 框架：{框架 / major 或 rendering decision；精確版本看 manifest}
- UI 套件：{套件}
- 狀態管理：{方案}（不使用 {替代方案}）
- HTTP：{客戶端}
- 路由：{方式}
- 建置：{工具}

## 基礎設施
- 部署：{CDN / 靜態主機}
- CI/CD：{工具}
```

### 全端
```markdown
# 技術棧
最後更新：YYYY-MM-DD

## 共通
- 後端 runtime：{主要版本或部署約束}
- 前端 runtime：{主要版本或語言約束}

## 後端
{...同純後端段...}

## 前端
{...同純前端段...}

## 基礎設施
{...}
```

### 手機
```markdown
# 技術棧
最後更新：YYYY-MM-DD

## 共通
- 語言 / runtime：{Swift / Kotlin / Dart / TS；精確 dependency version 以 manifest / lockfile 為準}
- 套件管理：{CocoaPods / Gradle / pub / npm}

## 行動端
- 平台：{iOS / Android / 跨平台}
- 框架：{原生 / RN / Flutter / Expo}
- 狀態管理：{方案}
- Local DB：{套件}
- 推播：{服務}
- 深層連結：{方式}

## 基礎設施
- 發布：{App Store / Play Store / TestFlight / Firebase}
- CI/CD：{工具}
- Crash 監控：{服務}
```
