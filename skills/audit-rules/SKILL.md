---
name: audit-rules
description: Audit `.agents/rules/` for coverage gaps, rule quality issues, template compliance, and source-of-truth readiness. Use periodically — before a release, after significant feature work, or when onboarding a new team member — or whenever rules may be stale, overweight, incomplete, redundant, or violating write-skill specs. Suggestions only — never modifies rules or source; use `maintain-rules` to apply findings. To check whether code violates current rules, use `check-rules` instead.
updated: 2026-05-29
version: 0.5.1
---

## Changelog

### 0.5.1 - 2026-05-29
- 新增 tech-stack version drift 判斷，區分 dependency 精確版本與可保留的 runtime / tooling 決策。
- 補充 data-model、deployment、todo 類 rules 的 anti-overreport guard，避免把必要摘要誤判為 overweight。

### 0.5.0 - 2026-05-27
- 新增 Source-of-Truth Readiness / Rule Weight 審查，判斷 rules 是否應收斂為 source/docs/tracker pointer。

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

審查 `.agents/rules/` 是否足以描述專案中穩定反覆出現的程式碼模式，並檢查 rules 檔本身是否清楚、可驗證、無重疊或衝突（Rule Quality）、是否符合各 write skill 定義的範本規格（Template Compliance），以及是否已經過肥、可改由 source/docs/tracker 作為事實來源（Source-of-Truth Readiness / Rule Weight）。

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

1. 全部（Coverage Gap + Rule Quality + Template Compliance + Source-of-Truth Readiness）（預設）
2. 只檢查 Coverage Gap
3. 只檢查 Rule Quality
4. 只檢查 Template Compliance
5. 只檢查 Source-of-Truth Readiness / Rule Weight
```

若使用者未指定，執行全部四個面向。

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

## Step 6：Source-of-Truth Readiness / Rule Weight 審查

此審查判斷 rules 是否承載了本該由 source code、正式 docs、runbook、issue tracker 或 changelog 承載的事實。它只做 evidence-based 診斷，不設計或修改 source code。

### 6.1 Source pointer candidate

當 rule 內的完整常數表、API/function 清單、schema/shape、部署項目或規則表已能由穩定 source 取得時，列為候選。
若 rule 已明確指向 source-of-truth，但仍複製完整 enum、allowlist、config table、function list、dependency version list 或 package version table，也列為候選；這代表已經有 pointer 但尚未完成收斂。

Anti-overreport：
- `data-model.md` 的 collection/table responsibility summary 若只描述用途、ownership、讀寫邊界，且 field-level shape 已指向 model/schema/source，不報 overweight。
- `deployment.md` 的 preview/prod 差異、production risk、rollback 限制、secret policy 即使已有 workflow/runbook pointer 也應保留，不因可指 source 就建議刪除。

可用 evidence：
- rule 提到的 path 存在，且該檔含集中 enum / registry / const / config / model / exported function
- 多處程式碼引用同一 source
- rule 已明確指向該 source
- 相關 source 近期 churn 低，或已有測試保護

Recommendation：建議用 `maintain-rules` 將 rule 收斂為摘要 + source pointer。

### 6.2 Tech-stack version drift

只針對 `tech-stack.md` 中已寫入的精確 dependency version、patch/minor version，或可由 manifest / workflow 驗證的 runtime version 做 evidence-based 檢查。

檢查規則：
- 若 `tech-stack.md` 已明確指向 manifest / lockfile，且沒有複製精確 dependency version，不報 drift。
- 若 rule 寫了 `package ^1.2.3`、`package 1.2.3`、完整 dependency version 清單，讀取可用 manifest（如 `pubspec.yaml`、`package.json`、`functions/package.json`、`go.mod`、lockfile）比對；不要求完整 semver resolver，只比對可直接證明的名稱與版本字串。
- 若 manifest 可讀且版本不一致，列為 Source pointer candidate 或 Rule Quality finding，Evidence 必須同時引用 `tech-stack.md` 行號與 manifest path。
- 若 manifest 不存在或無法可靠比對，不猜測 drift；建議改成 manifest / lockfile pointer。
- Node.js 22、ESM / `nodenext`、Flutter stable channel、package manager、部署 runtime 這類 runtime / tooling decision 可保留；audit 只檢查是否和 manifest、workflow 或 deploy config 明顯衝突。

Recommendation：建議用 `maintain-rules` 將精確 dependency version 收斂為 manifest pointer，並保留 runtime 決策與禁用替代方案。

### 6.3 Missing source of truth

當 rule 描述完整規則，但找不到單一穩定 source，或同一規則散落多檔時，列為缺 source of truth。

可用 evidence：
- rule 沒有 path，卻保存完整決策表 / 流程表 / API 契約
- 同一常數、規則或條件在多個 source 檔重複出現
- rule 與 source 有 drift
- 內容含「目前」「暫時」「待定」「Phase」且 source 尚未穩定

Recommendation：保留 rule，並輸出「需要 source refactor plan」；不要在 audit 內設計重構。

### 6.4 Temporary rule knowledge

開發中專案允許 rules 較厚。若內容明顯是過渡知識，但 source 還未穩定，列為暫時知識，不視為錯誤。

Recommendation：保留，並建議標記未來可收斂條件（例如 source 穩定、docs 補齊、tracker 建立）。

### 6.5 Backlog/history overweight

針對 `todo-and-plans.md` 或 planning 類 rule，檢查是否包含大量已完成項目、PR 編號、日期、release history 或長期 backlog。

Tracker detection：
- High：`.github/ISSUE_TEMPLATE/` 存在、GitHub Issues 近期活躍、README/AGENTS/docs 明確指向 GitHub Issues / Linear / Jira
- Medium：PR template 或 docs 要求 issue link
- Unknown：無本地或 GitHub evidence，不強推 tracker

Recommendation：若有 tracker evidence，建議遷移至 tracker；否則建議遷移 completed/history 至 `CHANGELOG.md`、`docs/history.md` 或 release notes。

若 `todo-and-plans.md` 只保留少量 In Progress、近期 Planned、Known Issues 或 Open Questions，不列為 backlog/history overweight；只有大量完成歷史、日期/PR 記錄、release history 或長期 backlog 才列 finding。

### 6.6 Overflow only

若內容仍屬 source 不容易快速推論的決策、邊界、禁忌、業務原因，且沒有明確 source/docs/tracker 可承接，只是篇幅過長，列為 Overflow only。

Recommendation：轉 `rules-overflow`。

所有 finding 必須附：
- **Evidence**：具體 rules 行號、source path、docs/tracker signal 或 grep 結果
- **Confidence**：High / Medium / Low
- **Recommendation**：`maintain-rules` 收斂、保留暫時知識、需要 source refactor plan、遷移 tracker/docs/changelog、或轉 `rules-overflow`

## Step 7：輸出報告

```
## audit-rules 報告
審查範圍：{Coverage Gap + Rule Quality + Template Compliance + Source-of-Truth Readiness / …}
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

### Source-of-Truth Readiness / Rule Weight

- {finding 標題}
  Evidence: `.agents/rules/{file}.md:{line}` + `{source-or-doc-path}`（若有）
  類型：{Source pointer candidate / Missing source of truth / Temporary rule knowledge / Backlog/history overweight / Overflow only}
  Confidence：{High / Medium / Low}
  現況：{rule 目前承載了什麼資訊}
  建議：{收斂成 source pointer / 保留暫時知識 / 需要 source refactor plan / 遷移 tracker 或 changelog / 轉 rules-overflow}

（若無，標示「無」）

### 建議後續

- 若有 Coverage Gap、跨檔搬移、缺檔、新鮮度、Rule Weight 或多類型 findings：執行 `maintain-rules`，它會承接本次 audit 報告並自動分類處理
- 若只有 Rule Quality / 失效引用等小範圍微修：仍建議使用 `maintain-rules`；進階使用者可直接使用 `update-rules`
- 若有 Missing source of truth：不要直接改 source，先建立 source refactor plan
- 若無 findings：無需修補
（若無，標示「無」）

### 摘要

發現 {gap_count} 個 coverage gap、{quality_count} 個 rule quality finding、{tc_count} 個 template compliance finding、{weight_count} 個 source-of-truth readiness finding（🔴 {red} / 🟠 {orange} / 🟢 {green} 通過）。
```

## 行為限制

- 只讀取、審查、報告；不直接修改 rules
- 不檢查程式碼是否違反既有 rules；這是 `check-rules` 的職責
- Coverage Gap 必須有至少 3 個檔案的反覆模式作為證據
- Rule Quality finding 必須引用具體 rules 檔路徑與行號；類型限於：模糊、不可驗證、重疊、衝突
- Template Compliance finding 必須引用具體行數、章節名或 git commit 數作為客觀依據；不憑語意判斷
- Source-of-Truth finding 必須附 Evidence、Confidence、Recommendation；不憑感覺建議重構
- 不設計或修改 source code；Missing source of truth 只輸出「需要 source refactor plan」
- 缺檔建議只列出缺失並引導至 `maintain-rules`，**不執行建檔**
- 失效引用檢查為可選項，預設不執行
- 若建議更新 rules，優先指向 `maintain-rules`；只有需要說明目標檔規格時才提及對應 write skill
- project-local rules 與一般 best practices 衝突時，優先尊重 project-local rules
