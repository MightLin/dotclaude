---
name: migrate-rules
description: Migrate existing projects from the old `.claude/rules/` layout to the shared `.agents/rules/` layout without rerunning init-project. Use when a project already has Claude rules, CLAUDE.md, or old dotclaude output and the user wants Codex compatibility, AGENTS.md creation, or low-token migration instead of regenerating project rules.
---

# Skill：遷移舊版 rules

## 目的
將已使用舊版 dotclaude 的專案遷移到 Claude / Codex 共用格式。
避免重新執行 init-project，保留既有 rules 內容與專案知識。

## 遷移目標

- 舊 rules：`.claude/rules/`
- 新 rules：`.agents/rules/`
- Claude 入口：`CLAUDE.md`
- Codex 入口：`AGENTS.md`

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

### 2. 建立或合併 `.agents/rules/`

- 若 `.agents/rules/` 不存在：建立目錄並複製 `.claude/rules/*`
- 若目標檔不存在：複製
- 若目標檔存在且內容相同：跳過
- 若目標檔存在且內容不同：不要覆蓋；列出檔名讓使用者決定

### 3. 更新入口檔

#### CLAUDE.md 存在
將其中的路徑：

- `.claude/rules/` → `.agents/rules/`
- `.claude\rules\` → `.agents\rules\`

不要改其他專案規則文字。

#### AGENTS.md 不存在
以 `CLAUDE.md` 為基礎建立精簡版：

- 標題可保留專案名稱
- 將「Claude」字樣改為「Codex」只限工具入口描述，不改業務規則
- 文件索引指向 `.agents/rules/`
- 保留核心原則與禁止事項

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

## 驗證

完成後檢查：

- `.agents/rules/` 存在
- `CLAUDE.md` 不再引用 `.claude/rules/`
- `AGENTS.md` 存在且引用 `.agents/rules/`
- `.agents/rules/` 內不再引用 `.claude/rules/`

## 輸出格式

```markdown
## 遷移摘要
- rules：{複製數} copied, {跳過數} unchanged, {衝突數} conflicts
- 入口檔：CLAUDE.md {updated/unchanged/missing}；AGENTS.md {created/updated/unchanged}
- 舊目錄：{保留/已刪除/不存在}

## 需要注意
- {若有衝突或未刪舊目錄，列出}
```
