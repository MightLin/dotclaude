---
name: rules-overflow
description: Internal sub-skill invoked by rules write skills (e.g. `architecture`, `business-logic`) when a planned write would exceed the file's line limit — offer the user a choice between compressing or splitting the file, with Claude-recommended options and reasons. Not intended for direct user invocation.
updated: 2026-05-25
version: 0.1.1
---

## Changelog

### 0.1.1 - 2026-05-25
- description 改為明示「Internal sub-skill，不適合使用者直接呼叫」

### 0.1.0 - 2026-05-23
- 建立初始 skill 規範。

# Skill：rules 超出行數上限處理

## 觸發條件

在寫入或更新任何 `.agents/rules/` 檔案**之前**，計算寫入後的預計行數。
若超過該 rules skill 規定的上限，停止寫入並進入本流程。

## Step 1：先查專屬建議

依目標 rules 檔判斷是否有領域特定的拆檔規則：

| rules 檔 | 優先建議 |
|---|---|
| `business-logic.md` | 依子領域拆成多個檔 |
| `design-guide.md` | 超出細節拆至 `.agents/design/<slug>/` |
| `todo-and-plans.md` | backlog 過多，建議遷移至 issue tracker |

若符合上表，先呈現專屬建議並附理由，詢問使用者是否採用。
例：「此檔超出上限通常代表領域過大，建議依子領域拆出 payment.md、subscription.md 等獨立檔。」
使用者拒絕或目標檔不在上表，進入 Step 2。

## Step 2：通用二選一（壓縮 / 分離）

向使用者呈現兩個選項，**必須附推薦選項與判斷依據**：

### 選項 A：壓縮

掃描現有內容，列出「可縮減」的候選區塊：

**推薦優先級（由高到低）**
1. 與其他 rules 檔重複的內容
2. 已過時或已不適用的條目
3. 範例可精簡（保留結構，刪多餘範例行）
4. 描述性段落（可改為一行摘要）

每個候選項附一句話說明為何可縮減。
使用者勾選確認後執行。

### 選項 B：分離

掃描現有內容，列出「相對低頻」的候選區塊：

**推薦優先級（由高到低）**
1. 只在特定情境查閱（如：新建模組、上線）
2. 進階或邊緣情境說明
3. 歷史性說明（已實作完成、純參考）
4. 可獨立成完整子主題的段落

每個候選項附一句說明為何適合分離。
使用者勾選確認後執行。

## Step 3：分離執行規則

命名：`.agents/rules/<原檔名 stem>-<slug>.md`（slug 用分離內容主題，kebab-case）

主檔新增一行索引（插入最後一個 `##` 段落之後、文件末尾，不插入 code block 內）：
```
- 低頻/進階：<主題> → ./<原檔名>-<slug>.md
```

分離檔頂部第一行：
```
分離自：./<原檔名>.md
```

分離檔**不繼承**原行數上限，但建議單檔 < 150 行（約為原上限的 2 倍，作為寬鬆緩衝）。

## 禁止行為

- 不在使用者確認前自動執行壓縮或分離
- 不分離被多個區塊共同引用的核心定義
- 不以壓縮為由刪除安全、合規、或邊界相關規則
