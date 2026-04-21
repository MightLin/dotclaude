# Skill：撰寫 api-conventions.md

## 目的
讓 Claude 產出的 API 程式碼符合專案規範，不會建議不一致的設計。

## 必要內容

### 1. URL 命名規則
### 2. HTTP 方法使用慣例
### 3. Request / Response 格式
### 4. 錯誤處理方式
### 5. 認證方式
### 6. 特殊規範

## 範例
```markdown
# API Conventions
最後更新：YYYY-MM-DD

## URL 規則
- 全小寫 kebab-case：/api/resource-name
- 資源用複數名詞
- 巢狀資源：/api/resources/{id}/sub-resources

## HTTP 方法
- GET：查詢（不改變資料）
- POST：新增
- PUT：整體更新
- PATCH：部分更新
- DELETE：刪除或作廢

## Response 格式
成功：直接回傳資料或 204 No Content
錯誤：{ code: string, message: string, details?: object }

## 錯誤處理
- 透過全域 middleware 統一處理，不在 Handler 內 try-catch

## 認證
- {認證方式說明}

## 特殊規範
- 列表 API 支援分頁：?page=1&pageSize=20
- 外部 API 呼叫統一透過封裝的 Service 層
```
