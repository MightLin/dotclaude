---
name: maintain-rules
description: Orchestrate project rules maintenance after audit-rules findings or after feature work. Use as the primary user-facing entry point to apply audit fixes, move misplaced rules, fill coverage gaps, refresh stale rules, or decide which `.agents/rules/` files should be updated from a feature description or git diff. Reads the relevant write-*-rules skills as rule-file specs before editing.
updated: 2026-05-27
version: 0.1.0
---

## Changelog

### 0.1.0 - 2026-05-27
- 建立 rules 維護總管 skill，承接 audit 報告與新功能後的 rules 更新。

# Skill：maintain-rules — rules 維護總管

## 目的

作為 rules 維護的主要入口，負責判斷、編排與套用 `.agents/rules/` 的最小必要更新。

此 skill 不取代各 `write-*-rules`；那些 skill 是各 rule 檔的寫作規格來源。需要修改某個 rule 檔前，必須先讀取對應 `write-*-rules` 的「必要內容」「禁止放入」「大小上限」。

## 適用場景

- 跑完 `audit-rules` 後，想直接套用修補
- audit 找到 Coverage Gap、放錯檔案、失效引用、Rule Quality 或 Template Compliance 問題
- 新功能完成後，不確定應更新哪些 `.agents/rules/`
- rules 可能因近期程式碼變更而過時，需要重新比對

不適合：只想檢查程式碼是否違反既有 rules；改用 `check-rules`。

## Step 1：判斷模式

依使用者請求選擇模式：

1. **Audit 修復模式**：使用者提到 audit findings、剛跑完 `audit-rules`、貼上 audit 報告，或要求修補 rules 問題。
2. **新功能更新模式**：使用者提到新功能、feature diff、這次變更後要補文件 / rules，或不知道該呼叫哪些 write skill。

若兩者都符合，先走 Audit 修復模式，再檢查新功能更新是否仍有未涵蓋的 rules。

## Step 2A：Audit 修復模式

取得最近一次或使用者貼上的 `audit-rules` 報告；若沒有報告，請使用者先執行 `audit-rules`。

將 findings 分類：

| 類型 | 處理方式 |
|---|---|
| Rule Quality（模糊 / 不可驗證 / 重疊 / 衝突） | 使用 `update-rules` 的手術式修補原則 |
| Template Compliance — 失效引用 / 可補章節 / 小型冗餘 | 手術式修補或補章節 |
| Template Compliance — 冗餘片段且應放到其他 rule | 語意搬移到目標 rule，再移除來源 |
| Coverage Gap | 依目標 rule 的 `write-*-rules` 規格補入正確檔案 |
| Freshness | 比對近期程式碼變更後決定是否刷新對應 rule |
| 行數超限 | 轉入 `rules-overflow` 流程 |

### 語意搬移規則

放錯檔案時，不可盲目 copy 原文。

1. 先抽出原段落真正表達的規則語意。
2. 讀取目標檔案對應的 `write-*-rules` 規格。
3. 依目標 rule 的章節、粒度、禁止項與既有風格重寫內容。
4. 若目標 rule 已有等價內容，合併或補強，不重複貼上。
5. 確認目標 rule 已保存語意後，才移除來源段落。

只有原文已精準、簡短、完全符合目標 rule 寫法時，才可近似原文搬移；仍須通過目標 `write-*-rules` 檢查。

## Step 2B：新功能更新模式

讀取使用者提供的 feature 描述、變更檔案清單或 `git diff`。若使用者沒有指定範圍，優先檢查目前 git diff。

依變更內容決定候選 rules：

| 變更線索 | 候選 rule |
|---|---|
| API route / controller / handler / client / SDK call | `api-conventions.md` |
| schema / migration / entity / model / repository | `data-model.md` |
| domain service / state flow / permission / billing / calculation | `business-logic.md` |
| package / framework / runtime / tooling / env convention | `tech-stack.md` |
| component / style / theme / layout / interaction pattern | `design-guide.md` |
| test helper / test layout / coverage / E2E / fixture | `testing-strategy.md` |
| CI / deployment / Docker / release / secret handling | `deployment.md` |
| MCP server / tool / resource / prompt / host policy | `mcp-conventions.md` |
| roadmap / known issue / planned work | `todo-and-plans.md` |

只更新因本次變更產生「可重複遵守的專案慣例」的 rules。不要把單次實作細節、完整 API 清單、完整 schema 或一般 best practices 寫進 rules。

## Step 3：建立修補清單

輸出即將處理的工作清單，包含：

- 目標 rule 檔
- 來源 evidence（audit finding、檔案路徑、diff 片段或使用者描述）
- 更新類型（新增、替換、語意搬移、刪除、補章節、建立缺檔）
- 會讀取的 `write-*-rules` 規格
- 可能的 overflow 風險

若需要建立不存在的 rule 檔，先讀 `init-project` 的專案類型對照表；仍不確定是否適用時，詢問使用者。

## Step 4：套用更新

對每個目標 rule：

1. 讀取目標 rule 檔與對應 `write-*-rules`。
2. 擬定最小修改，不整份重寫。
3. 確認不違反目標 skill 的「禁止放入」。
4. 計算寫入後行數；若超過上限，使用 `rules-overflow`。
5. 若 front matter 有 `updated` 欄位，更新為今天日期（`YYYY-MM-DD`）。
6. 對跨檔搬移，先寫入或合併目標，再刪除來源。

## Step 5：驗證與輸出

完成後輸出：

```markdown
## maintain-rules 完成

更新：
- `.agents/rules/{file}.md`：{更新原因與摘要}

搬移：
- `{source}` → `{target}`：{語意搬移摘要}

略過：
- {略過原因}

建議驗證：
- {建議執行 check-rules / audit-rules 的範圍}
```

## 行為限制

- 不整份重寫 rules；只做最小必要段落新增、替換、搬移或刪除
- 不直接修改程式碼；只維護 `.agents/rules/`
- 不把 `write-*-rules` 合併進本 skill；它們永遠是目標檔規格來源
- 不把 audit 報告中的文字盲目 copy 到目標檔
- 不在目標 rule 保存語意前刪除來源段落
- project-local rules 與一般 best practices 衝突時，優先尊重 project-local rules
