---
name: mcp-conventions
description: Write or update project MCP design rules. Use when initializing or maintaining `.agents/rules/mcp-conventions.md` for MCP server design, tool naming, tool description, input schema, Resource/Tool/Prompt selection, side-effect tiering, MCP host integration, or projects using @modelcontextprotocol/sdk / FastMCP.
updated: 2026-05-10
version: 0.1.0
---

## Changelog

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：撰寫 mcp-conventions.md

## 目的
讓 agent 修改 MCP server 或 host 整合程式碼時，遵循專案既有設計準則（tool 粒度、description 風格、schema、副作用分級），避免產出 LLM 難用或破壞性的工具介面。

## 適用範圍
- 必要：build / 維護 MCP server 的專案
- 條件：MCP client / host，且有自家工具選用、授權、後處理慣例
- 不需要：純 client 且無自家整合慣例、完全不碰 MCP

## 必要內容（依角色挑用）

### 服務端段（提供 MCP server）

#### 1. Primitive 選擇
- Tool：model 可主動呼叫的操作 / 查詢 / 計算，包含 read-only 與有副作用行為
- Resource：host/client 可納入上下文的 URI 定址資料，通常無副作用
- Prompt：user/host 可選用的可重用提示模板，不存放長期專案規則
- 一句話判斷準則寫進 rule

#### 2. 命名與粒度
- 命名風格（snake_case / camelCase、動詞-名詞）
- prefix / namespace（避免多 server 撞名）
- 寧可多個小工具，不要單一大工具帶 mode 參數
- Resource URI 設計（scheme、層級）

#### 3. Description 撰寫
- 受眾是 LLM，不是人類開發者
- 必含：何時用、何時不用、典型輸入線索
- 避免：實作細節、版本資訊、SDK 用語

#### 4. Input schema
- 必填 vs 選填判斷標準
- 每個欄位描述語意（給 LLM 判讀）
- enum 優於自由字串
- 攤平 vs 巢狀的取捨

#### 5. 回傳格式
- 純文字 / 結構化 content 的選用
- 大型輸出策略：截斷、回傳檔案路徑、分頁
- 錯誤訊息含讓 LLM 自我修正的線索

#### 6. 副作用分級（強制）
- read-only：純讀取，無狀態變更
- write：建立 / 更新資源
- destructive：刪除 / 不可逆操作
- destructive 工具必須有 confirmation 慣例（host 提示 / 雙步驟 / dry-run）

#### 7. Auth / scope / 多租戶
- 認證機制（API Key / OAuth / Session）
- Scope 設計與最小權限
- 多租戶隔離欄位

#### 8. Logging / Telemetry
- 在哪一層記錄 tool 呼叫
- 敏感參數遮罩規則

### 附錄：MCP host/client 整合方時補寫
當專案負責 MCP host/client 整合，且有自家工具選用、授權、後處理慣例時加入：
- 預設啟用 vs 需授權的工具分類
- 失敗 / 逾時 / retry 策略
- Tool 結果後處理層歸屬
- 多 server 命名衝突處理（前綴 / 別名）

## 禁止放入
- 完整 tool 清單（屬程式碼）
- 個別 tool 的完整 schema（屬程式碼 / SDK 自動產出）
- SDK 教學或 quickstart（屬 README）

## 大小上限
產出檔案不超過 80 行。
超出時請使用 rules-overflow skill 與使用者協作決定壓縮或分離。

## 範例

```markdown
# MCP Conventions
最後更新：YYYY-MM-DD

## Primitive 選擇
- model 需要主動觸發的操作 / 查詢 / 計算 → Tool
- host/client 可納入上下文的 URI 定址資料 → Resource
- user/host 可選用的可重用提示模板 → Prompt

## 命名
- tool：snake_case，動詞-名詞（list_orders、create_invoice）
- 全部 tool 加前綴 {project}_，避免 host 多 server 撞名
- Resource URI：{scheme}://{type}/{id}

## 粒度
- 一個 tool 做一件事，不接受 mode / action 切換用途
- 超過 6 個必填參數要 review

## Description
- 第一句寫「何時用」、第二句寫「何時不用」
- 不寫實作細節、不寫版本

## Input schema
- enum 優於自由字串
- 巢狀只在語意成群時使用
- 每欄位 description 不少於一句完整話

## 回傳
- 預設純文字 content
- 超過 {N} KB 改回傳檔案路徑或 resource URI
- 錯誤含 actionable hint

## 副作用分級
- read-only：list_*, get_*, search_*
- write：create_*, update_*
- destructive：delete_*, drop_*；必須回 confirmation token，第二次呼叫才執行

## Auth
- {API Key via header / OAuth scope}
- 多租戶：{tenant_id 從 session 帶入，禁止 tool 參數}
```
