---
name: write-deployment-rules
description: Write or update deployment rules. Use when initializing or maintaining `.agents/rules/deployment.md` for environments, secrets sources, CI/CD triggers, deployment targets, rollback, migrations, mobile store release flows, monitoring, logging, and alerting.
updated: 2026-05-27
version: 0.3.0
---

## Changelog

### 0.3.0 - 2026-05-27
- 新增 Source of Truth 原則，區分部署風險規則與應移至 workflow/runbook/changelog 的完整操作內容。

### 0.2.0 - 2026-05-27
- skill 改名為 `write-deployment-rules`，讓 skill 名稱描述撰寫/維護 rules 的動作，並保留產出檔名不變。

### 0.1.1 - 2026-05-23
- 大小上限段新增超出時使用 rules-overflow skill 的提示。

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：撰寫部署 rules

## 目的
讓 Claude 知道部署目標、環境設定來源、發布流程，避免把開發 / 測試環境的設定寫進程式或 PR。

## Source of Truth 原則

- rules 保存部署風險、環境差異、secret 管理、migration/rollback 原則與高風險提醒。
- 完整 CI yaml、完整 function 清單、完整部署命令或一次性操作歷史應指向 workflow、runbook、README 或 changelog。
- 開發中專案可暫時把部署 runbook 放在 rule；流程穩定後應遷移至 docs/runbook，rule 只保留摘要與 pointer。

## 適用範圍
- 必要：所有專案類型
- 不需要：純 library（用 npm publish / NuGet 即可，無多環境問題）

## 必要內容（依專案類型挑用）

### 共通段
- 環境清單（dev / stage / prod 或更細）
- 環境變數來源（.env 檔 / Vault / SSM / Doppler / GitHub Secrets）
- CI/CD 工具與觸發條件（push 哪個 branch、tag、PR 標籤）
- 回滾方式（重跑舊 image / 切流量 / DB 還原）

### 服務段（Web / API / Backend）
- 部署目標（Cloud Run / Fargate / k8s / IIS / Heroku / ...）
- 容器化（Dockerfile 位置、image registry）
- 健康檢查端點
- Migration 在部署中的位置（先跑 / 同時跑）

### 靜態前端段
- 部署目標（Vercel / Netlify / S3+CloudFront / Pages）
- Build 指令與輸出路徑
- 預覽環境（PR preview）

### 手機 app 段
- iOS：TestFlight → App Store 流程、簽章管理（fastlane / Match）
- Android：Internal → Closed → Open → Production 流程、簽章管理
- Code push / OTA（若用 Expo / CodePush）
- 版本號與 build 號規則

### 監控段（共通）
- Logging 平台
- Metrics / APM
- 告警通道
- 錯誤追蹤（Sentry / Crashlytics）

## 禁止放入
- 任何實際 secret 值
- 完整 CI yaml（屬程式碼）
- 部署過的歷史紀錄（屬 release notes）

## 大小上限
產出檔案不超過 60 行。
超出時請使用 rules-overflow skill 與使用者協作決定壓縮或分離。

## 範例

### Web / API
```markdown
# Deployment
最後更新：YYYY-MM-DD

## 環境
- dev / stage / prod

## 環境變數
- 來源：{Vault / SSM / GitHub Environment}
- 必要變數清單：{KEY_LIST}（值不寫在這）

## CI/CD
- 工具：{GitHub Actions}
- 觸發：push main → stage；tag v* → prod
- Migration：deploy 前先跑

## 部署目標
- {Cloud Run / k8s namespace}
- Image：{registry path}
- 健康檢查：GET /healthz

## 回滾
- {ArgoCD rollback / 重 deploy 上一個 tag}

## 監控
- Logs：{Loki / CloudWatch}
- APM：{Datadog / New Relic}
- 告警：{Slack #alerts}
```

### 手機
```markdown
# Deployment
最後更新：YYYY-MM-DD

## 版本號
- iOS / Android 共用 marketing version；build 號獨立遞增

## iOS
- TestFlight：每 PR merge 自動發布
- App Store：手動 promote
- 簽章：{fastlane Match}，憑證在 {位置}

## Android
- Internal track：每 PR merge
- Production：手動 promote
- 簽章：{Play App Signing}

## OTA（若使用）
- {Expo Updates / CodePush} channel：{清單}
- 規則：JS-only 變更才用 OTA，原生變更必須走 store

## 監控
- Crash：{Crashlytics / Sentry}
- 分析：{Amplitude / Mixpanel}
```
