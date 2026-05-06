# Skill：撰寫 api-conventions.md

## 目的
讓 Claude 產出的 API 程式碼（提供端或消費端）符合專案規範。

## 適用範圍
- 必要：backend / fullstack（提供 API），frontend / mobile（消費 API 且有自家規範）
- 不需要：純 library、純 CLI、無外部 API 互動

## 必要內容（依角色挑用）

### 服務端段（提供 API）
依採用的風格挑：

#### REST
- URL 命名（kebab-case / camelCase、複數名詞、巢狀規則）
- HTTP 方法用法
- Request / Response 格式（JSON envelope vs 直接物件）
- 分頁 / 排序 / 過濾 query string 慣例
- 錯誤格式（code / message / details）
- 認證 header
- 版本策略（URL 前綴 / Header）

#### GraphQL
- Schema 命名規範
- Query / Mutation 命名前綴
- Pagination（Connection / offset）
- 錯誤回傳（top-level errors vs union type）
- 認證機制

#### gRPC / RPC
- proto 檔案組織
- 方法命名
- 錯誤碼策略
- Streaming 使用情境

#### 共通
- 認證方式（JWT / OAuth / API Key / Session）
- Rate limit / Idempotency
- 錯誤處理位置（global middleware vs handler 內）

### 客戶端段（消費 API）
- HTTP 客戶端封裝慣例（Service 層 / Repository 層）
- 錯誤處理（哪一層攔截、如何呈現給 UI）
- 重試 / 逾時 / 取消策略
- 認證 token 儲存與更新
- 離線快取策略（手機尤其重要）

## 禁止放入
- 完整 endpoint 清單（屬 OpenAPI / 程式碼）
- DTO 欄位規範（屬資料模型 / schema）
- 範例 payload（屬 API 文件）

## 大小上限
產出檔案不超過 80 行。

## 範例

### REST 服務端
```markdown
# API Conventions
最後更新：YYYY-MM-DD

## 風格
REST + JSON

## URL
- 全小寫 kebab-case：/api/resource-name
- 複數名詞、巢狀：/api/resources/{id}/sub-resources
- 版本：/api/v1/...

## HTTP 方法
- GET / POST / PUT / PATCH / DELETE 對應 CRUD

## Response
- 成功：直接物件或 204
- 錯誤：{ code, message, details? }

## 錯誤處理
- 透過全域 middleware 統一處理，不在 handler try-catch

## 認證
- {Bearer JWT / Session Cookie}

## 列表
- 分頁：?page=1&pageSize=20
- 排序：?sort=field,-otherField
```

### 客戶端
```markdown
# API Conventions（消費端）
最後更新：YYYY-MM-DD

## 來源
- {後端服務名}：{base URL}（環境變數 {KEY}）

## 客戶端封裝
- 統一透過 {Repository / Service} 層呼叫，禁止 component / view 直接呼叫
- HTTP 客戶端：{套件}

## 錯誤處理
- 401：{重新登入流程}
- 5xx：{重試策略}
- 業務錯誤：{依 code 對應使用者訊息}

## Token
- 儲存：{安全儲存機制}
- 更新：{refresh 流程}

## 離線（手機）
- 快取策略：{stale-while-revalidate / cache-first}
- 同步機制：{背景同步觸發點}
```
