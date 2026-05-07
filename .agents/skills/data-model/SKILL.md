---
name: data-model
description: Write or update data model rules. Use when initializing or maintaining `.agents/rules/data-model.md` for database type, naming conventions, primary keys, timestamps, soft delete, indexes, migrations, sensitive fields, or local mobile database rules.
---

# Skill：撰寫 data-model.md

## 目的
讓 Claude 修改資料層時遵循專案既有慣例（命名、主鍵、軟刪除、migration 流程），避免破壞 schema 一致性。

## 適用範圍
- 必要：backend / fullstack
- 條件：mobile（有 local DB：SQLite / Realm / Room / Core Data）
- 不需要：純前端、純 stateless service

## 必要內容

### 1. 資料庫類型與選型
- 主資料庫（PostgreSQL / MySQL / MongoDB / DynamoDB / ...）
- 副資料庫 / 快取（Redis / Memcached）
- 為何選此型（簡述）

### 2. 命名規範
- 表 / 集合：單數 vs 複數、大小寫、前綴
- 欄位：大小寫、布林前綴（is_/has_）、時間欄位（created_at vs createdAt）
- 主鍵：自增 / UUID / ULID
- 外鍵命名

### 3. 共通欄位
- 時間戳記（created_at / updated_at / deleted_at）
- 軟刪除策略（是否使用、如何過濾）
- 多租戶欄位（若有）
- 樂觀鎖 / 版本欄位

### 4. 索引策略
- 預設索引方針
- 覆蓋索引使用情境
- 不建索引的時機

### 5. Migration 流程
- 工具（EF Migrations / Alembic / Flyway / Prisma / TypeORM / ...）
- 命名規則
- Breaking change 處理（雙寫期、相容性）
- 在 PR 與部署的順序

### 6. 敏感欄位
- PII 加密欄位清單方式
- 密碼 / token 雜湊規範

## 禁止放入
- 完整 schema 定義（屬程式碼 / migrations）
- 個別表的欄位清單（除非有非顯而易見的語意）
- ER 圖（屬獨立文件）

## 大小上限
產出檔案不超過 70 行。

## 範例

### 關聯式（後端）
```markdown
# Data Model
最後更新：YYYY-MM-DD

## 資料庫
- 主：{PostgreSQL 16}
- 快取：{Redis 7}

## 命名
- 表：複數 snake_case（users, order_items）
- 欄位：snake_case
- 主鍵：bigserial 或 UUID v7
- 外鍵：{referenced_table}_id

## 共通欄位
- created_at / updated_at（timestamptz，預設 now()）
- deleted_at（軟刪除，查詢需過濾）

## 索引
- 所有外鍵自動建索引
- 高頻查詢條件加複合索引，超過 3 欄要 review

## Migration
- 工具：{Alembic / Flyway}
- 命名：{YYYYMMDD_HHMM__description}
- Breaking change：先加新欄位 → 雙寫 → 切換讀 → 移除舊欄位

## 敏感欄位
- 密碼：bcrypt cost 12
- PII：{欄位清單} 使用 pgcrypto AES-256
```

### Mobile（local DB）
```markdown
# Data Model
最後更新：YYYY-MM-DD

## 資料庫
- {SQLite via Room / Core Data / Realm}

## 命名
- {依框架慣例}

## 共通欄位
- updated_at：用於與 server 同步
- sync_status：local-only / pending / synced

## Migration
- {Room AutoMigration / 手寫 schema version 升級}
- 不允許破壞性 migration（必要時提供匯出再匯入流程）

## 同步策略
- {last-write-wins / version vector}
- 衝突解決：{方式}

## 敏感欄位
- token：{Keychain / Keystore}，不存 DB
```
