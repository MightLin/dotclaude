呼叫 Stitch MCP 產生 UI 設計，subagent 自評通過後輸出 design-guide.md。

1. 確認 `.claude/rules/` 已就緒（至少有 architecture.md 或 business-logic.md）。若無，建議使用者先執行 `/init-project`。
2. 觸發 `stitch-design` skill，進入收集輸入 → orchestration loop → 終止流程。
3. PASS 後詢問使用者是否將 design-guide 草稿寫入 `.claude/rules/design-guide.md`。
4. 若 3 次仍 FAIL，呈現分數歷史與最後一版設計，請使用者選擇接受 / 調整 brief 重跑 / 中止。
