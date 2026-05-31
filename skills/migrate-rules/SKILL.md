---
name: migrate-rules
description: Migrate existing projects from the old `.claude/rules/` layout to the shared `.agents/rules/` layout, or upgrade an existing entrypoint file (CLAUDE.md / AGENTS.md) to the current `entrypoint-writing` template (e.g. fill in missing core principles). Use when a project already has Claude rules, CLAUDE.md, or old dotclaude output and the user wants Codex compatibility, AGENTS.md creation, low-token migration instead of regenerating project rules, or just bringing an outdated entrypoint up to the current template without redoing init-project.
updated: 2026-06-01
version: 0.4.0
---

## Changelog

### 0.4.0 - 2026-06-01
- 遷移後以 `AGENTS.md` 作為入口 source of truth；`CLAUDE.md` 改為薄轉址並提醒內容應寫到 `AGENTS.md`。

### 0.3.1 - 2026-05-28
- 釐清建立 `AGENTS.md` 時的入口檔語意，避免把核心原則與禁止事項誤寫成 `.agents/rules/` 的內容。

### 0.3.0 - 2026-05-22
- 新增 Step 3.5：升級入口檔核心原則到當前 `entrypoint-writing` 範本
- description 擴充涵蓋已遷移專案的入口檔升級場景

### 0.2.0 - 2026-05-14
- 新增 Step 6：清理全域遺留的舊 commands（sync.md、init-project.md）。

### 0.1.0 - 2026-05-10
- 建立初始 skill 規範。

# Skill：遷移舊版 rules

## 目的
將已使用舊版 dotclaude 的專案遷移到 Claude / Codex 共用格式。
避免重新執行 init-project，保留既有 rules 內容與專案知識。

## 遷移目標

- 舊 rules：`.claude/rules/`
- 新 rules：`.agents/rules/`
- Claude 入口：`CLAUDE.md`（薄轉址）
- Codex / 主入口：`AGENTS.md`

## 原則

- 先複製，確認後才刪除舊檔。
- 不重寫 rules 內容；只更新路徑與入口檔。
- 若 `.agents/rules/` 已存在且內容不同，停下來列出衝突並詢問。
- 若只有 `.claude/rules/` 存在，不重新掃描技術棧、不重新產生 rule 檔。
- 完成前列出遷移摘要與剩餘風險。

## 步驟

### 1. 掃描現況

檢查：

- `.claude/rules/` 是否存在
- `.agents/rules/` 是否存在
- `CLAUDE.md` 是否存在
- `AGENTS.md` 是否存在

只列檔名與大小；不要一開始讀完整 rules。

掃描完成後，列出本次預計執行的步驟與將異動的檔案，等待使用者確認後再進入 Step 2。本 skill 會修改入口檔與 rules，不應在使用者不知情下執行。

### 2. 建立或合併 `.agents/rules/`

- 若 `.agents/rules/` 不存在：建立目錄並複製 `.claude/rules/*`
- 若目標檔不存在：複製
- 若目標檔存在且內容相同：跳過
- 若目標檔存在且內容不同：不要覆蓋；列出檔名讓使用者決定

### 3. 更新入口檔

入口檔以 `AGENTS.md` 作為 source of truth；`CLAUDE.md` 只指向 `AGENTS.md`。

#### AGENTS.md 存在
將其中的路徑：

- `.claude/rules/` → `.agents/rules/`
- `.claude\rules\` → `.agents\rules\`

不要改其他專案規則文字。

#### AGENTS.md 不存在
以 `CLAUDE.md` 為基礎建立精簡版：

- 標題可保留專案名稱
- 將「Claude」字樣改為「Codex」只限工具入口描述，不改業務規則
- 文件索引指向 `.agents/rules/`
- 保留入口檔內的核心原則與禁止事項；不要寫成 `.agents/rules/` 的核心原則或禁止事項

若沒有 `CLAUDE.md`，建立最小 `AGENTS.md`：

```markdown
# {ProjectName}

## 文件索引
- 架構：.agents/rules/architecture.md
- 技術棧：.agents/rules/tech-stack.md

## 核心原則
1. 接到任務前，view 相關 rules 檔
2. 開新功能前，先讀 todo-and-plans.md（若存在）
3. 開始實作前，列出預計異動的檔案範圍
```

只列實際存在的 rules。

#### CLAUDE.md

確認 `AGENTS.md` 存在後，建立或改寫為薄轉址：

```markdown
# {ProjectName}

本專案入口規則以 `AGENTS.md` 為主；Claude Code 開始作業前請先讀 `AGENTS.md`。

所有原本要寫進 `CLAUDE.md` 的內容，都應改寫到 `AGENTS.md`，避免兩份入口規則分歧。
```

不要在 `CLAUDE.md` 複製文件索引、核心原則或禁止事項。

### 3.5 升級入口檔核心原則

對 `AGENTS.md`，view `skills/entrypoint-writing/SKILL.md` 取得當前「必含的核心原則」清單，與入口檔內「## 核心原則」段比對：

1. 列出**缺少的條目**（以條目語意比對，不只比字串；例如「完成修改後主動建議跑 /check-rules」這條，無論語句長短只要語意涵蓋即視為已存在）
2. 在「## 核心原則」段內**只插入缺少的條目**，編號順延後面的專案特有規則
3. 不動文件索引、不動禁止事項、不動其他客製內容
4. 若入口檔找不到「## 核心原則」段或結構與範本差異過大，停下來列出差異讓使用者決定是否人工合併

此步驟對「從舊 `.claude/rules/` 遷移過來」與「已是 `.agents/rules/` 但想升級入口檔」兩種情境都適用 — 前者由 Step 2/3 帶進來，後者使用者直接呼叫此 skill 即會走到這步（Step 2 會偵測無 `.claude/rules/` 而跳過搬移）。`CLAUDE.md` 不執行核心原則升級，只維持指向 `AGENTS.md`。

### 4. 更新 rules 內部路徑

搜尋 `.agents/rules/` 下的舊路徑字串：

- `.claude/rules/`
- `.claude\rules\`

若出現，做路徑替換。不要改其他 `.claude/` 字串，因為它可能指 Claude 專用設定。

### 5. 保留或清理舊 rules

完成複製與入口更新後，詢問使用者是否刪除 `.claude/rules/`。

- 使用者同意：刪除 `.claude/rules/`
- 使用者未同意：保留，並在摘要中註明舊目錄仍存在

不要刪除 `.claude/agents/`、`.claude/settings.local.json`。

### 6. 清理全域遺留的舊 commands

檢查 `~/.claude/commands/` 是否有舊版 dotclaude 遺留的 commands：

- `sync.md`：舊版 sync，已被 `sync-config` skill 取代
- `init-project.md`：舊版 init-project，參照舊路徑（`~/.claude/skills/claude-md-writing/`），已被 `init-project` skill 取代

若存在，提示使用者可以刪除，並提供指令讓使用者自行執行：

```powershell
Remove-Item "$HOME\.claude\commands\sync.md" -Force
Remove-Item "$HOME\.claude\commands\init-project.md" -Force
```

不要自動刪除，讓使用者確認後自行執行。

## 驗證

完成後檢查：

- `.agents/rules/` 存在
- `CLAUDE.md` 指向 `AGENTS.md`，且不複製文件索引、核心原則或禁止事項
- `AGENTS.md` 存在且引用 `.agents/rules/`
- `.agents/rules/` 內不再引用 `.claude/rules/`

## 輸出格式

```markdown
## 遷移摘要
- rules：{複製數} copied, {跳過數} unchanged, {衝突數} conflicts
- 入口檔：AGENTS.md {created/updated/unchanged}；CLAUDE.md {redirect-created/redirect-updated/missing}
- 入口檔核心原則：{已是最新/插入 N 條}
- 舊目錄：{保留/已刪除/不存在}

## 需要注意
- {若有衝突或未刪舊目錄，列出}
```
