---
name: write-testing-strategy-rules
description: Write or update testing strategy rules. Use when initializing or maintaining `.agents/rules/testing-strategy.md` for test frameworks, test pyramid, coverage, CI test execution, mock boundaries, integration tests, E2E tests, mobile UI tests, or visual regression.
updated: 2026-05-27
version: 0.2.0
---

## Changelog

### 0.2.0 - 2026-05-27
- skill 改名為 `write-testing-strategy-rules`，讓 skill 名稱描述撰寫/維護 rules 的動作，並保留產出檔名不變。

### 0.1.1 - 2026-05-23
- 大小上限段新增超出時使用 rules-overflow skill 的提示。

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：撰寫測試策略 rules

## 目的
讓 Claude 知道專案怎麼測試、什麼該測、什麼不該 mock，避免寫出無效測試或破壞 CI。

## 適用範圍
- 必要：所有專案類型
- 不需要：純文件倉、純設定倉

## 必要內容（依專案類型挑用）

### 共通段
- 測試框架（單元 / 整合 / e2e 各用什麼）
- 覆蓋率目標（若有）
- 測試金字塔分配（哪一層佔比多）
- CI 上的執行策略（PR 跑哪些、main merge 跑哪些）
- Mock 邊界：什麼可以 mock、什麼必須真實（最常踩雷的點）

### 後端段（backend / fullstack）
- 整合測試是否使用真實 DB（推薦用 testcontainers / docker-compose）
- 外部 API 的 mock 方式（VCR / wiremock / fake server）
- Migration / seed 在測試環境的處理

### 前端段（frontend / fullstack）
- Component test：框架（Vitest + Testing Library / Jest / ...）
- E2E：工具（Playwright / Cypress）
- Visual regression（若有）
- Mock API 的方式（MSW / fixtures）

### 行動端段（mobile）
- UI test / instrumentation test（XCUITest / Espresso / Detox / Maestro）
- 單元測試框架（XCTest / JUnit）
- 真機 vs 模擬器策略
- 測試裝置矩陣（OS 版本、螢幕尺寸）

## 禁止放入
- 個別測試案例的詳細描述（屬程式碼）
- 完整測試清單（屬程式碼）
- 框架使用教學（屬框架文件）

## 大小上限
產出檔案不超過 60 行。
超出時請使用 rules-overflow skill 與使用者協作決定壓縮或分離。

## 範例

### 後端
```markdown
# Testing Strategy
最後更新：YYYY-MM-DD

## 框架
- 單元：{框架}
- 整合：{框架} + 真實 DB（testcontainers）
- E2E：{若有}

## 金字塔
- 單元 70% / 整合 25% / E2E 5%

## 覆蓋率
- Service / Domain 層 ≥ 80%
- Handler 層不強制覆蓋率，靠整合測試保護

## Mock 邊界
- 禁止 mock：DB、ORM
- 必須 mock：外部第三方 API
- 工具：{wiremock / VCR / ...}

## CI
- PR：unit + integration
- main：全部 + e2e
```

### 前端
```markdown
# Testing Strategy
最後更新：YYYY-MM-DD

## 框架
- 單元 / Component：Vitest + {Testing Library}
- E2E：{Playwright / Cypress}

## 金字塔
- Component 70% / E2E 30%（不寫純函式單元測試除非邏輯複雜）

## Mock 邊界
- API：用 MSW，不在 component 內 mock fetch
- 路由：使用真實 router，不 mock

## CI
- PR：component + 關鍵 e2e
- main：完整 e2e 矩陣
```

### 手機
```markdown
# Testing Strategy
最後更新：YYYY-MM-DD

## 框架
- 單元：{XCTest / JUnit}
- UI test：{XCUITest / Espresso / Detox / Maestro}

## 裝置矩陣
- iOS：{版本清單}
- Android：{版本清單}

## Mock 邊界
- 網路層：{方式}
- Local DB：使用 in-memory 替身
- 推播：mock 接收器

## CI
- PR：unit + 主要 UI test 在模擬器
- Release：真機矩陣
```
