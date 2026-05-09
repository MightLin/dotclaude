# Stitch Design Rubric

## 門檻
- 總分（加權）≥ 80
- 任一維度 ≥ 70
- 上述兩者皆滿足才視為 PASS

## 硬性 FAIL
任一情況發生時，直接 FAIL：

- 未確認 mode 就呼叫 Stitch。
- brief 缺必要資訊仍生成。
- `feature-extension` 未分析既有風格來源。
- Stitch 設計階段沒有提供 project link，且未請 user 決定 fallback。
- user 尚未確認要實作，就下載/export 或產生 implementation handoff。

## Mode-aware 維度

### greenfield

| 維度 | 權重 | 評分要點 |
|---|---:|---|
| 需求覆蓋 | 35% | 是否涵蓋產品目的、目標使用者、平台、核心功能與主要畫面；缺漏關鍵工作流直接 < 70 |
| 資訊架構 / 流程 | 25% | 頁面結構、導航、主要 user flows 是否清楚且符合任務優先級 |
| 可用性 / accessibility | 20% | 資訊層級、操作路徑、對比度、鍵盤可操作、ARIA/語意是否合理 |
| 實作可行性 | 20% | 是否能用一般前端技術與專案可能的 UI 套件實作，避免罕見元件或複雜動畫依賴 |

### feature-extension

| 維度 | 權重 | 評分要點 |
|---|---:|---|
| 需求符合 | 25% | 是否解決新增功能的目標，入口、流程、影響頁面與關鍵操作是否完整 |
| 既有風格一致 | 30% | 是否依據 design-guide、現有頁面、截圖或程式碼延伸色彩、字級、間距、元件與互動模式 |
| 接合方式 / 狀態完整 | 20% | 是否說清楚與既有 IA/頁面/元件的接合方式，並涵蓋 empty/loading/error/success/disabled 等狀態 |
| 實作可行性 | 25% | 是否符合專案 UI 套件、技術限制與禁止事項，可交給 Claude Code 實作 |

### design-guide-refresh

| 維度 | 權重 | 評分要點 |
|---|---:|---|
| 規範完整 | 35% | 是否涵蓋 UI 套件、佈局、元件慣例、表單、互動、responsive、accessibility 與禁止事項 |
| 專案一致 | 30% | 是否忠實反映既有專案設計、技術棧與產品型態，而非產生通用模板 |
| 可操作性 | 20% | 規則是否足以指導後續實作，避免空泛形容詞 |
| 簡潔維護性 | 15% | 是否精簡、易掃描、避免重複與過度細節 |

## 計分

依 mode 選擇對應權重：

```text
total = Σ(score_i × weight_i)
```

每維度 0–100。Feedback 必須具體到 generator 下一輪可做出可觀察改變。
