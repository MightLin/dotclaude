# Skill：撰寫 tech-stack.md

## 目的
讓 Claude 知道用什麼技術、版本、慣用套件，避免給出不符合現況的建議。

## 必要內容

### 後端
- 語言與版本
- 框架與版本
- ORM 與資料庫
- 慣用套件（列出用途）
- 不使用的替代方案（避免 Claude 建議錯誤的套件）

### 前端
- 框架與版本
- UI 套件
- 狀態管理方式
- HTTP 客戶端

### 基礎設施
- 部署環境
- CI/CD（若有）
- 其他重要服務

## 範例
```markdown
# 技術棧
最後更新：YYYY-MM-DD

## 後端
- 語言：C# .NET 8
- 框架：ASP.NET Core Minimal API
- ORM：EF Core 8 + SQL Server
- HTTP 客戶端：Refit（不使用 HttpClient 直接呼叫）
- 物件映射：Mapster（不使用 AutoMapper）
- 背景服務：IHostedService + IDbContextFactory

## 前端
- 框架：Angular 19
- UI 套件：PrimeNG 17
- 架構：Signal-based（OnPush + computed/effect）
- HTTP：Angular HttpClient 封裝為 Service

## 基礎設施
- 部署：IIS on Windows Server
- 資料庫：SQL Server（本機）
- 郵件：IIS 內建 SMTP
```
