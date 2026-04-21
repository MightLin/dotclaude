# Skill：撰寫 architecture.md

## 目的
讓 Claude 快速理解系統的組成、各模組的職責邊界、資料流向。

## 必要內容

### 1. 系統概述（3 句以內）
專案是什麼、服務誰、核心價值。

### 2. 模組清單
每個模組一行，格式：
`- ModuleName：職責說明（相關檔案路徑或命名空間）`

### 3. 資料流（可用文字描述或 ASCII 圖）
主要業務流程的資料怎麼流動：
`使用者操作 → Component → Service → API → Backend Handler → DB`

### 4. 重要邊界規則
哪些模組不應該直接互相依賴、哪些是共用核心。

## 範例
```markdown
# 系統架構
最後更新：YYYY-MM-DD

## 系統概述
{一句話說明系統目的與服務對象}

## 模組清單
- ModuleA：職責說明（/features/module-a/）
- ModuleB：職責說明（/features/module-b/）
- SharedService：共用服務（/services/）

## 資料流
使用者操作 → Component → Service → API Handler → EF Core → SQL Server

## 邊界規則
- Feature 模組不直接呼叫其他 Feature 模組，透過 Service 層溝通
- 外部 API 呼叫只允許在 /services/ 層發起
```
