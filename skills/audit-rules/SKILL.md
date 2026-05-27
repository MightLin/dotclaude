---
name: audit-rules
description: Audit `.agents/rules/` for coverage gaps, rule quality issues, and template compliance. Use periodically — before a release, after significant feature work, or when onboarding a new team member — or whenever rules may be stale, incomplete, redundant, or violating write-skill specs. Suggestions only — never modifies rules; use `maintain-rules` to apply findings. To check whether code violates current rules, use `check-rules` instead.
updated: 2026-05-27
version: 0.4.0
---

## Changelog

### 0.4.0 - 2026-05-27
- 建議後續改為優先導向 `maintain-rules`，避免使用者自行串接多個 write skills。

### 0.3.1 - 2026-05-25
- description 補充「定期使用」場景（release 前 / 大功能後 / onboarding 時）與 `check-rules` cross-reference

### 0.3.0 - 2026-05-23
- 失效引用詢問移至 Step 2，僅在審查範圍含 Template Compliance 時才詢問
- 新鮮度白名單補全七個標準 rule 檔的對應目錄
- 非標準 rules 檔（無對應 write skill）在 Template Compliance 中明確標示「無範本基準」
- Step 1 移除「可能的維護問題」（判斷留給後續 Step）
- 摘要行加入 Template Compliance 問題計數

### 0.2.0 - 2026-05-23
- 新增 Template Compliance 審查面向（行數使用率、可補章節、缺檔偵測、新鮮度、失效引用）
- Step 2 選單擴充為四選項，加入「只檢查 Template Compliance」
- Rule Quality 移除「過長」「放錯檔案」兩類（改歸 Template Compliance，有客觀依據）

### 0.1.0 - 2026-05-23
- 建立 rules 覆蓋缺口與 rules 品質審查 skill

# Skill：audit-rules — 規則覆蓋與品質審查

## 目的

審查 `.agents/rules/` 是否足以描述專案中穩定反覆出現的程式碼模式，並檢查 rules 檔本身是否清楚、可驗證、無重疊或衝突（Rule Quality），以及是否符合各 write skill 定義的範本規格（Template Compliance）。

此 skill 只產出審查報告與維護建議；不直接修改 rules。若要檢查程式碼是否違反既有 rules，改用 `check-rules` skill。

## 適用範圍

- 必要：專案已有 `.agents/rules/` 目錄（否則建議先執行 `init-project`）
- 適合：定期 rules 維護、專案導入後校準、發現 rules 與實作脫節時
- 不適合：PR 合規檢查、單純找程式碼違規
- 掃描範圍：永遠對整個專案全域掃描，不支援 diff 模式。若要在 diff 範圍內做合規檢查，改用 `check-rules`。

## Step 1：讀取 rules 現況

列出 `.agents/rules/` 下所有 `.md` 檔。若該目錄不存在，中止並提示：「尚未初始化規則，請先執行 init-project skill。」

並行讀取所有 rules 檔，整理每個檔案的主題、可查核規則與描述性段落。

## Step 2：確認審查範圍

詢問使用者要審查的範圍：

```
請選擇 audit-rules 的審查範圍：

1. 全部（Coverage Gap + Rule Quality + Template Compliance）（預設）
2. 只檢查 Coverage Gap
3. 只檢查 Rule Quality
4. 只檢查 Template Compliance
```

若使用者未指定，執行全部三個面向。

若審查範圍含 Template Compliance（選項 1 或 4），額外詢問：
「本次是否也要做失效引用檢查？（需 grep 全專案，成本較高，預設否）」

## Step 3：Coverage Gap 審查

掃描專案程式碼，找出反覆出現但 `.agents/rules/` 未涵蓋的模式。

Coverage Gap 審查永遠以整個專案為範圍（非 diff 範圍），因為需要跨 ≥3 個檔案確認反覆模式是否形成專案級共識。

掃描整個專案時，自動排除：
- `node_modules/`, `vendor/`, `.git/`, `dist/`, `build/`, `.next/`, `.nuxt/`
- 二進位檔、圖片、`.lock` 檔

Coverage Gap 必須同時符合：

1. 同類模式出現在至少 3 個檔案
2. `.agents/rules/` 完全未提及該模式
3. 能明確歸屬到某個既有 rule 檔主題，或能建議新增哪類 rule 檔

不要把一般 best practices 當成缺口。只有 repo 程式碼已經存在穩定反覆模式，且 rules 完全未描述時，才列為 gap。

## Step 4：Rule Quality 審查

Rule Quality 審查只讀 `.agents/rules/` 檔本身，不掃程式碼。

審查 rules 檔本身的**語意品質**，記錄具體問題：

- 模糊：規則無法指導實作，例如「保持乾淨」但沒有可觀察標準
- 不可驗證：無法從程式碼、設定或流程檢查是否遵守
- 重疊：多個 rules 檔描述同一規範，且可能造成維護分歧
- 衝突：不同 rules 對同一行為提出互斥要求

Rule Quality finding 必須引用具體 rules 檔路徑與行號；不需要引用程式碼位置。

> 「過長」與「放錯檔案」有客觀範本依據（write skill 的「大小上限」與「禁止放入」清單），改由 Step 5 Template Compliance 審查。

## Step 5：Template Compliance 審查

Template Compliance 以**客觀範本與 git 對照**為主，補足 Rule Quality 語意審查的不足。

### 5.1 前置（自動執行）

- 並行讀取每個現有 rules 檔對應的 write skill（rule 檔 basename 對應 `skills/write-{basename}-rules/SKILL.md`，例如 `architecture.md` → `skills/write-architecture-rules/SKILL.md`），提取「大小上限」「必要內容」「禁止放入」三段
  - 若某 rules 檔找不到對應的 write skill（自訂 rule 檔），**略過**行數使用率、冗餘片段、可補章節三項，在報告中標示「無範本基準，跳過」
- 讀取 `skills/init-project/SKILL.md` 取得專案類型 × rule 對照表
- 依 `init-project` Step 3 線索自動偵測專案類型：
  - frontend：`package.json` + 前端框架且無後端框架
  - backend：`*.csproj` / `pom.xml` / `go.mod` / `requirements.txt` / `Gemfile` 等且無前端 entry
  - fullstack：同時有前端與後端框架，或 server/client 雙資料夾
  - mobile：`build.gradle` + `AndroidManifest.xml`、`*.xcodeproj`、`pubspec.yaml`、Expo / React Native
  - 無法判斷 → 跳過缺檔比對，並在報告中標示「類型未明」
- 失效引用是否執行：依 Step 2 使用者回答決定；未回答則預設不執行

### 5.2 分項檢查

- **行數使用率**：取 rules 檔的實際行數，除以對應 write skill 的「大小上限」；≥90% 標 🟠、>100% 標 🔴、其餘 🟢
- **冗餘片段**：以「禁止放入」清單的關鍵字搜尋 rules 檔內容，記錄命中區段與行號，建議移到正確 rule 檔
- **可補章節**：對照「必要內容」清單，若 rules 檔內找不到對應的 `##` 標題，列為缺漏
- **新鮮度（相對 git 活躍度，避免絕對日期失真）**：
  - 取 rules 檔的最後 commit 日期 `R`（`git log -1 --format=%cI -- <file>`）
  - 取**同主題程式碼**從 `R` 之後的 commit 數 `C`，依以下白名單對應：
    | rule 檔 | 對應程式碼目錄 / 檔案 |
    |---|---|
    | `architecture.md` | `src/` |
    | `tech-stack.md` | `package.json` / `go.mod` / `requirements.txt` / `*.csproj` / `Gemfile` |
    | `data-model.md` | migration 目錄（`migrations/` / `db/migrate/`） |
    | `api-conventions.md` | routes / controllers / handlers 目錄 |
    | `business-logic.md` | services / domain / usecases 目錄 |
    | `design-guide.md` | components / styles / themes 目錄 |
    | `testing-strategy.md` | `tests/` / `__tests__/` / `spec/` |
    | `deployment.md` | `.github/workflows/` / `Dockerfile` / `docker-compose.yml` |
    | `mcp-conventions.md` | mcp / server / tools 目錄 |
    | `todo-and-plans.md` | （無對應程式碼目錄，跳過新鮮度檢查） |
  - 若對應目錄不存在於 repo，跳過該檔的新鮮度檢查
  - `C ≥ 30` 標 🟠、`C ≥ 100` 標 🔴；`C = 0`（repo 本身沒動）不誤報
- **缺檔偵測**：依自動偵測的專案類型，比對 `init-project` 表格找出尚未建立的 rule 檔；**只列出缺失，不執行建檔**，建議使用者執行 `maintain-rules` 判斷是否建立
- **失效引用**（可選，需使用者確認才執行）：對 rules 中出現的相對路徑、檔名做 `git ls-files` 比對，列出已不存在的引用

## Step 6：輸出報告

```
## audit-rules 報告
審查範圍：{Coverage Gap + Rule Quality + Template Compliance / …}
Rules 檔：{實際讀取的 rules 檔清單}
偵測專案類型：{frontend / backend / fullstack / mobile / 類型未明}

### Coverage Gaps

- {反覆模式標題}
  Evidence: `{path1}`、`{path2}`、`{path3}`（至少 3 個檔案）
  現況：{程式碼中反覆出現的模式}
  缺口：`.agents/rules/` 未涵蓋 {主題}
  建議：執行 `maintain-rules` 承接本 finding；它會讀取 `{對應 write skill 名稱}` 作為目標檔規格

（若無，標示「無」）

### Rule Quality Findings

- {品質問題標題}
  Evidence: `.agents/rules/{file}.md:{line}`
  類型：{模糊 / 不可驗證 / 重疊 / 衝突}
  現況：{rule 目前怎麼寫}
  建議：{應如何調整，並指向 `maintain-rules`；必要時註明目標 write skill 規格}

（若無，標示「無」）

### Template Compliance

| 檔案 | 行數 / 上限 | 使用率 | 新鮮度（C 值） | 狀態 |
|---|---|---|---|---|
| {file}.md | {n} / {max} | {n/max}% | {同步 C=0 / 🟠 C=N / 🔴 C=N} | {🟢/🟠/🔴} |

**冗餘片段**（命中「禁止放入」清單）
- `.agents/rules/{file}.md:{line}` — 命中「{item}」，建議移到 `{target-rule}.md`

（若無，標示「無」）

**可補章節**（缺少「必要內容」中的章節）
- `.agents/rules/{file}.md` — 缺章節「{section}」（來自 `{write-skill}` skill 的必要內容）

（若無，標示「無」）

**缺檔建議**
- 依偵測到的專案類型（{type}），尚可考慮補：`{file}.md`
  建議：執行 `maintain-rules` 判斷是否建立，並讀取 `{write-skill}` 作為目標檔規格

（若類型未明或無缺檔，標示「無」）

**失效引用**（若有執行）
- `.agents/rules/{file}.md:{line}` 引用 `{path}` 已不存在於 `git ls-files`

（若無或未執行，標示「未執行」或「無」）

### 建議後續

- 若有 Coverage Gap、跨檔搬移、缺檔、新鮮度或多類型 findings：執行 `maintain-rules`，它會承接本次 audit 報告並自動分類處理
- 若只有 Rule Quality / 失效引用等小範圍微修：仍建議使用 `maintain-rules`；進階使用者可直接使用 `update-rules`
- 若無 findings：無需修補
（若無，標示「無」）

### 摘要

發現 {gap_count} 個 coverage gap、{quality_count} 個 rule quality finding、{tc_count} 個 template compliance finding（🔴 {red} / 🟠 {orange} / 🟢 {green} 通過）。
```

## 行為限制

- 只讀取、審查、報告；不直接修改 rules
- 不檢查程式碼是否違反既有 rules；這是 `check-rules` 的職責
- Coverage Gap 必須有至少 3 個檔案的反覆模式作為證據
- Rule Quality finding 必須引用具體 rules 檔路徑與行號；類型限於：模糊、不可驗證、重疊、衝突
- Template Compliance finding 必須引用具體行數、章節名或 git commit 數作為客觀依據；不憑語意判斷
- 缺檔建議只列出缺失並引導至 `maintain-rules`，**不執行建檔**
- 失效引用檢查為可選項，預設不執行
- 若建議更新 rules，優先指向 `maintain-rules`；只有需要說明目標檔規格時才提及對應 write skill
- project-local rules 與一般 best practices 衝突時，優先尊重 project-local rules
