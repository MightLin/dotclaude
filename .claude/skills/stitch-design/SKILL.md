# Skill：呼叫 Stitch MCP 產生 UI 設計

## 目的
依系統目的與功能，呼叫 Stitch MCP 取得 UI 設計與 `design-guide.md` 草稿；自評通過 rubric 才交付。低分自動帶 feedback 重試，最多 3 次。

## 流程

### 1. 收集輸入
依序執行：
1. view `.agents/rules/architecture.md`（若存在）→ 取系統目的、模組
2. view `.agents/rules/business-logic.md`（若存在）→ 取主要功能
3. view `.agents/rules/design-guide.md`（若存在）→ 作為視覺一致性比對基準
4. 缺項才向使用者詢問：
   - 系統目的（一句話）
   - 主要功能列表
   - 目標使用者
   - 平台（Web / mobile / both）

整理為 brief 區塊（格式見 stitch-generator）。

### 2. Orchestration Loop（最多 3 次）

```
attempt = 1
feedback = none

while attempt <= 3:
    gen_output = Agent(subagent_type="stitch-generator",
                       prompt=brief + (feedback if feedback else ""))

    if gen_output starts with "ERROR:":
        中止，回報錯誤給使用者
        break

    eval_output = Agent(subagent_type="stitch-evaluator",
                        prompt=brief + gen_output + 既有 design-guide)

    解析 eval_output 取得分數、PASS/FAIL、feedback
    記錄此輪結果到歷史

    if PASS:
        break
    else:
        feedback = eval_output 的 Feedback 區塊
        attempt += 1
```

### 3. 終止
- **PASS**：呈現設計摘要與 `design-guide.md` 草稿給使用者，詢問是否寫入 `.agents/rules/design-guide.md`。
- **FAIL × 3**：呈現所有輪次的分數歷史與最後一版設計，請使用者選擇：
  - 接受最後一版
  - 調整 brief 重跑
  - 中止

## 注意

- 兩個 subagent 都不會把 MCP 的大量原始 payload 回吐給主執行緒，只回摘要與分數。
- 主執行緒不要重新評分，信任 evaluator 結果。
- rubric 可調：使用者若有偏好可暫時改 `rubric.md` 的權重或門檻，主執行緒不需修改。
