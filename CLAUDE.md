# Global Rules

## 核心行為原則

- 接到任務前，view 專案 `.claude/rules/` 下相關檔案以理解現況，不憑空假設
- 開新功能前，先讀 `.claude/rules/todo-and-plans.md`（若存在）確認與計畫無衝突
- 開始實作前，列出預計異動的檔案範圍給使用者確認
- 修改程式前先 view 相關檔案，不整份重寫
- 輸出簡潔，推理完整
- 已讀檔案不重讀，除非可能已被修改
- 超過 100KB 檔案除非必要略過，可詢問是否需要
- 宣告完成前先測試程式

## 全域 Commands 說明

- /init-project：首次導入新專案，建立標準 .claude/ 結構
- /sync：將 dotclaude repo 同步到 ~/.claude/
- /stitch-design：呼叫 Stitch MCP 產生 UI 設計，subagent 自評通過後輸出 design-guide.md
