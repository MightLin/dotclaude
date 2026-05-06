專案導入助手。依序執行，不跳過。

## Step 1：讀取操作知識

view ~/.claude/skills/project-structure/SKILL.md
view ~/.claude/skills/claude-md-writing/SKILL.md

## Step 2：掃描專案現況

- view 專案根目錄結構
- 確認是否已有 CLAUDE.md 或 .claude/ 目錄
- 若已有，先讀現有內容

## Step 3：偵測專案類型

掃描下列線索推斷類型：

- **frontend**：`package.json` + 前端框架（react/vue/angular/svelte/solid）+ 無後端框架
- **backend**：`*.csproj` / `pom.xml` / `go.mod` / `requirements.txt` / `Gemfile` / `package.json` + 後端框架（express/fastify/nestjs），且無前端 entry
- **fullstack**：同時有前端框架與後端框架，或單一專案有 server/client 雙資料夾
- **mobile**：`build.gradle` + `AndroidManifest.xml`、`*.xcodeproj` / `Info.plist`、`pubspec.yaml`（Flutter）、`app.json` + `expo`（Expo）、`metro.config.js`（RN）

無法判斷時直接詢問。推斷後向使用者確認。

## Step 4：偵測技術棧

掃描設定檔（套件管理、框架、容器、CI），自行讀取。
偵測後只問**無法從檔案推斷**的部分：

- 專案目的與服務對象
- 主要功能模組
- 業務領域是否有特殊術語 / 流程
- TODO 來源（issue tracker？或要建 todo-and-plans.md？）

## Step 5：條件式建立 rules 檔

依專案類型決定建哪些（參考 ~/.claude/skills/project-structure/SKILL.md）：

| Rule 檔 | frontend | backend | fullstack | mobile |
|---|---|---|---|---|
| architecture.md | ✓ | ✓ | ✓ | ✓ |
| tech-stack.md | ✓ | ✓ | ✓ | ✓ |
| business-logic.md | 視專案 | 視專案 | 視專案 | 視專案 |
| todo-and-plans.md | ✓（除非用 tracker） | ✓ | ✓ | ✓ |
| testing-strategy.md | ✓ | ✓ | ✓ | ✓ |
| deployment.md | ✓ | ✓ | ✓ | ✓ |
| api-conventions.md | 視有無自家規範 | ✓ | ✓ | 視有無自家規範 |
| design-guide.md | ✓ | ✗ | ✓ | ✓ |
| data-model.md | ✗ | ✓ | ✓ | 有 local DB 才建 |

撰寫每個檔案時，view 對應的 skill 取得格式與大小上限：

- ~/.claude/skills/architecture/SKILL.md
- ~/.claude/skills/tech-stack/SKILL.md
- ~/.claude/skills/business-logic/SKILL.md
- ~/.claude/skills/todo-and-plans/SKILL.md
- ~/.claude/skills/testing-strategy/SKILL.md
- ~/.claude/skills/deployment/SKILL.md
- ~/.claude/skills/api-conventions/SKILL.md
- ~/.claude/skills/design-guide/SKILL.md
- ~/.claude/skills/data-model/SKILL.md

## Step 6：建立 CLAUDE.md

依 ~/.claude/skills/claude-md-writing/SKILL.md 規範建立精簡 CLAUDE.md。
- 文件索引段只列實際建出來的 rules
- 必含 3 條核心原則（取代過去的 /understand、/new-feature 行為）

## Step 7：確認

列出已建立檔案，詢問是否需補充或修正。
