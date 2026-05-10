# dotclaude — Repo Architecture

## 系統概述
此 repo 是 Claude Code / Codex 的全域設定來源，透過 sync-config skill 一次性複製到本機全域位置。
所有其他專案共用的 skills 和行為原則均從這裡管理與發布。

## 目錄結構與職責

- `CLAUDE.md`：全域 Claude 入口；同步後成為 `~/.claude/CLAUDE.md`；只放核心行為原則
- `AGENTS.md`：全域 Codex 入口；同步後成為 `~/.codex/AGENTS.md`；只放核心行為原則 + Codex 路徑對應
- `skills/`：全域 skills 來源；同步到 `~/.claude/skills/` 與 `~/.codex/skills/`
- `.agents/skills/sync-config/`：Codex repo-local skill，僅在此 repo 內執行同步流程
- `.claude/skills/sync-config/`：Claude Code repo-local wrapper，轉發至 `.agents/skills/sync-config/`
- `.claude/agents/`：Claude Code 專用 subagents（stitch-generator、stitch-evaluator）；未同步到全域
- `.agents/rules/`：此 repo 自身的專案知識，不會同步出去

## 資料流（sync-config）

dotclaude repo → sync-config skill → `~/.claude/CLAUDE.md`、`~/.claude/skills/*`、`~/.codex/AGENTS.md`、`~/.codex/skills/*`

## Skill metadata 規範

- 所有 root `skills/*/SKILL.md` 都必須在 front matter 包含 `updated: <UTC YYYY-MM-DD>` 與 `version: <semver>`
- 所有 root `skills/*/SKILL.md` 都必須包含 `## Changelog`，且只保留最新一版作為同步前參考
- 修改 root skill 內容時必須同步更新 `updated`；重大流程或行為變更時同步調整 `version`
- Review skills 變更時，必須檢查 metadata 與 changelog 是否存在且符合本規範

## 新增 Skill 的流程

1. 在 `skills/<skill-name>/SKILL.md` 建立 skill 定義
2. 依照「Skill metadata 規範」補齊 `updated`、`version` 與 `## Changelog`
3. 若需要 Codex repo-local 版本，同時建立 `.agents/skills/<skill-name>/SKILL.md`
4. 若需要 Claude Code repo-local wrapper，建立 `.claude/skills/<skill-name>/SKILL.md`
5. 執行 sync-config skill 將變更同步到全域位置

## 邊界規則

- `CLAUDE.md` / `AGENTS.md` 只放全域行為原則；repo 特有說明放在本檔（`.agents/rules/architecture.md`）
- `.agents/rules/` 不會被 sync-config 同步出去，只作用於此 repo
- `.claude/settings.local.json` 不同步（含個人 API key / 本機路徑）
- `.claude/agents/` 目前尚未定義跨工具同步格式，不同步
- 不要在 root CLAUDE.md / AGENTS.md 中描述 skill 清單；skills 透過 Claude 的 skill 系統自動浮現
