---
name: init-project
description: Initialize a project for shared Claude/Codex agent guidance. Use when onboarding a new project without existing `.agents/rules/`, generating rule files, creating `CLAUDE.md` or `AGENTS.md`, detecting frontend/backend/fullstack/mobile project type, or replacing old dotclaude initialization workflows.
updated: 2026-05-29
version: 0.1.2
---

## Changelog

### 0.1.2 - 2026-05-29
- 修正初始化完成後的驗證建議：`init-project` 自行確認入口檔索引與 rules 建立結果；`check-rules` 改為後續程式碼修改後 / PR 前使用。

### 0.1.1 - 2026-05-14
- 建立入口檔前需詢問使用者要建立 `CLAUDE.md`、`AGENTS.md` 或兩者。

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：專案導入

依序執行，不跳過。若發現舊版 `.claude/rules/`，改用 `migrate-rules` skill，不重跑完整 init。

## Step 1：讀取操作知識

view `skills/entrypoint-writing/SKILL.md`

## Step 2：掃描專案現況

- view 專案根目錄結構
- 確認是否已有 `CLAUDE.md`、`AGENTS.md` 或 `.agents/rules/` 目錄
- 若已有舊版 `.claude/rules/`，停止本流程並改用 `migrate-rules`
- 若已有入口檔或 rules，先讀現有內容

## Step 3：偵測專案類型

掃描下列線索推斷類型：

- frontend：`package.json` + 前端框架（react/vue/angular/svelte/solid）+ 無後端框架
- backend：`*.csproj` / `pom.xml` / `go.mod` / `requirements.txt` / `Gemfile` / `package.json` + 後端框架，且無前端 entry
- fullstack：同時有前端框架與後端框架，或單一專案有 server/client 雙資料夾
- mobile：`build.gradle` + `AndroidManifest.xml`、`*.xcodeproj` / `Info.plist`、`pubspec.yaml`、Expo / React Native 線索

無法判斷時直接詢問。推斷後向使用者確認。

## Step 4：偵測技術棧

掃描設定檔（套件管理、框架、容器、CI），自行讀取。
偵測後只問無法從檔案推斷的部分：

- 專案目的與服務對象
- 主要功能模組
- 業務領域是否有特殊術語 / 流程
- TODO 來源（issue tracker？或要建 todo-and-plans.md？）
- 是否會 build / 維護 MCP server，或整合多個 MCP server 並維護自家授權、工具選用、後處理慣例（線索：`@modelcontextprotocol/sdk`、`mcp` 套件、`mcp-server*` 檔名、server entry 內含 MCP SDK import、MCP client/host 設定、tool allowlist、server registry）

## Step 5：條件式建立 rules 檔

rules 一律建立在 `.agents/rules/`。依專案類型決定建哪些：

| Rule 檔 | frontend | backend | fullstack | mobile |
|---|---|---|---|---|
| architecture.md | yes | yes | yes | yes |
| tech-stack.md | yes | yes | yes | yes |
| business-logic.md | 視專案 | 視專案 | 視專案 | 視專案 |
| todo-and-plans.md | yes（除非用 tracker） | yes | yes | yes |
| testing-strategy.md | yes | yes | yes | yes |
| deployment.md | yes | yes | yes | yes |
| api-conventions.md | 視有無自家規範 | yes | yes | 視有無自家規範 |
| design-guide.md | yes | no | yes | yes |
| data-model.md | no | yes | yes | 有 local DB 才建 |
| mcp-conventions.md | 視專案 | 視專案 | 視專案 | 視專案 |

`mcp-conventions.md` 不依 frontend/backend 軸判斷。獨立詢問：build / 維護 MCP server，或當 host/client 但有自家授權、工具選用、後處理慣例 → 必建；純消費（只用別人的 MCP server，無自家整合慣例）→ 不建。

撰寫每個檔案時，view 對應 skill 取得格式與大小上限：

- write-architecture-rules
- write-tech-stack-rules
- write-business-logic-rules
- write-todo-and-plans-rules
- write-testing-strategy-rules
- write-deployment-rules
- write-api-conventions-rules
- write-design-guide-rules
- write-data-model-rules
- write-mcp-conventions-rules

## Step 6：建立入口檔

依 entrypoint-writing skill 規範建立精簡入口檔。

- 建立前先詢問使用者要建立 `CLAUDE.md`、`AGENTS.md` 或兩者
- 若使用者未指定，依本次對話使用的工具給預設建議，但仍需等使用者確認
- 文件索引段只列實際建出來的 rules
- 必含 3 條核心原則（dotclaude 流程特有，取代過去的 /understand、/new-feature 行為；這些原則不在全域設定，只在有導入此流程的專案入口）

## Step 7：自行確認與完成報告

初始化完成後先自行確認，不要求使用者立即執行 `check-rules`：

- `CLAUDE.md` / `AGENTS.md` 的文件索引只列實際存在的 `.agents/rules/` 檔案
- 建立的 rules 檔符合 Step 3 偵測出的專案類型與 Step 5 條件式建立表
- 不存在的 rule 檔若被略過，完成報告需列出略過原因（例如純前端不建 `data-model.md`、已有 tracker 不建厚 `todo-and-plans.md`）

完成報告需列出：

- 已建立或更新的入口檔
- 已建立或更新的 rules 檔
- 明確略過的 rules 與原因
- 後續建議：後續修改程式碼後或 PR 前再執行 `check-rules`；若懷疑 rules 覆蓋不足、過時或過肥，執行 `audit-rules`，再用 `maintain-rules` 承接修補

不要在剛初始化完成時要求使用者立刻跑 `check-rules` 來確認 rules 落差；rules 本身的覆蓋與品質問題屬 `audit-rules` 職責。
