---
name: init-project
description: Initialize a project for shared Claude/Codex agent guidance. Use when onboarding a new project without existing `.agents/rules/`, generating rule files, creating `CLAUDE.md` or `AGENTS.md`, detecting frontend/backend/fullstack/mobile project type, or replacing old dotclaude initialization workflows.
---

# Skill：專案導入

依序執行，不跳過。若發現舊版 `.claude/rules/`，改用 `migrate-rules` skill，不重跑完整 init。

## Step 1：讀取操作知識

view `skills/project-structure/SKILL.md`
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

撰寫每個檔案時，view 對應 skill 取得格式與大小上限：

- architecture
- tech-stack
- business-logic
- todo-and-plans
- testing-strategy
- deployment
- api-conventions
- design-guide
- data-model

## Step 6：建立入口檔

依 entrypoint-writing skill 規範建立精簡入口檔。

- Claude 專案建立 `CLAUDE.md`
- Codex 專案建立 `AGENTS.md`
- 若使用者明確要雙工具共用，兩個都建
- 文件索引段只列實際建出來的 rules
- 必含 3 條核心原則（取代過去的 /understand、/new-feature 行為）

## Step 7：確認

列出已建立檔案，詢問是否需補充或修正。
