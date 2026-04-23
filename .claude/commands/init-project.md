專案導入助手。依序執行，不跳過。

## Step 1：讀取操作知識

view ~/.claude/skills/project-structure/SKILL.md
view ~/.claude/skills/claude-md-writing/SKILL.md

## Step 2：掃描專案現況

- view 專案根目錄結構
- 確認是否已有 CLAUDE.md 或 .claude/ 目錄
- 若已有，先讀現有內容

## Step 3：自動偵測技術棧

掃描根目錄及子目錄，找技術棧相關設定檔（套件管理、框架、環境、容器等），自行讀取。

偵測後只問**無法從檔案推斷**的部分：

- 專案目的與服務對象
- 主要功能模組
- 有無特殊 API 或 UI 規範
- 最重要的 TODO

## Step 4：建立文件結構

建立：

- .claude/rules/architecture.md（參考 ~/.claude/skills/architecture/SKILL.md）
- .claude/rules/tech-stack.md（參考 ~/.claude/skills/tech-stack/SKILL.md）
- .claude/rules/design-guide.md（有 UI 規範時）（參考 ~/.claude/skills/design-guide/SKILL.md）
- .claude/rules/api-conventions.md（有 API 規範時）（參考 ~/.claude/skills/api-conventions/SKILL.md）
- .claude/rules/todo-and-plans.md（參考 ~/.claude/skills/todo-and-plans/SKILL.md）
- .claude/rules/business-logic.md（參考 ~/.claude/skills/business-logic/SKILL.md）
- .claude/commands/understand-project.md

## Step 5：建立 CLAUDE.md

依 ~/.claude/skills/claude-md-writing/SKILL.md 規範建立精簡 CLAUDE.md。

## Step 6：確認

列出已建立檔案，詢問是否需補充或修正。
