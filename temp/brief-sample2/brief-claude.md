---
slug: tw-futures-gamified
generated: 2026-05-16
mode: greenfield
---

# Design Brief: tw-futures-gamified
產生時間：2026-05-16
Mode：greenfield

## 產品概覽
- **目的**: 台灣期貨紙上交易 + 交易日誌 App，讓嚴肅零售交易者在無風險環境中練習、紀錄、反思、成長
- **目標使用者**: 嚴肅零售交易者（台灣市場，熟悉期貨操作）
- **平台**: Flutter，Mobile 優先（375px 基準），Web 為 bonus

## 功能範圍
- **主要功能**: 即時模擬下單（大台/小台）、部位管理、5分鐘反思機制、交易日誌、統計分析、遊戲化排行榜
- **核心流程**: 登入 → 看報價 → 下單 → 持倉監控 → 交易後反思 → 日誌瀏覽 → 統計回顧 → 排名競爭
- **主要畫面（共 6+1）**:
  1. Auth — 登入/註冊（Email、Google、匿名）
  2. Trading — 即時報價 + 下單（主畫面）
  3. Positions — 當前持倉 + 未實現 P&L
  4. Journal — 交易日誌列表 + 反思問卷 + 詳情頁
  5. Stats — P&L 曲線、MDD、勝率、風報比圖表
  6. Ranking — 全球排行榜 + 等級 + 成就系統
  7. Reflection — 交易後5分鐘結構化反思（modal/overlay）

## 視覺方向
- **調性**: 遊戲化活潑（Gamified Dark），帶強烈霓虹感，兼顧交易介面的資訊密度；參考 Discord Dark + TradingView + 手遊 HUD
- **主色調（品牌色）**: 電紫色 `#7C3AED`（按鈕、頁籤 active、高亮），輔色青藍 `#06B6D4`（次要資訊、等級標籤）
- **語義色**: 漲/買 `#22C55E`（綠），跌/賣 `#EF4444`（紅），台灣市場慣例
- **背景層**:
  - 最底層: `#0F0F1A`
  - 卡片: `#1A1A2E`
  - 分隔/elevated: `#252540`
- **等級色彩系統**: Bronze `#CD7F32` / Silver `#C0C0C0` / Gold `#FFD700` / Platinum `#E5E4E2` / Diamond `#B9F2FF`
- **互動狀態需求**:
  - Loading: Shimmer skeleton（不用 CircularProgressIndicator 在卡片內）
  - Empty: 插畫 + CTA 說明
  - Error: SnackBar（輕量）/ Dialog（需確認）
- **黑名單**: 不用亮色主題、不用 Material You 彩色系統、不用 Segmented Control 做下單方向（用大型 Buy/Sell 按鈕）、不用 BottomSheet 做下單表單

## 技術限制
- **UI 套件 / 框架**: Flutter + fl_chart（圖表唯一選擇）+ google_fonts
- **狀態管理**: flutter_bloc（不用 Provider / Riverpod / GetX）
- **間距系統**: 8dp 倍數（8、16、24、32）
- **導航**: BottomNavigationBar 5頁（Trading / Positions / Journal / Stats / Ranking）
- **不可違反**: 不在 Widget 層直接呼叫 API 或 Repository；不使用亮色主題

---

## 給 AI 設計工具的 Prompt

```
Design a complete dark-themed mobile app UI for "TW Futures Trainer" — a Taiwan futures paper trading and trading journal app for serious retail traders.

**Visual Direction:** Gamified dark theme. Think Discord Dark meets TradingView with light game HUD elements. Dense information layout with vibrant neon accents. NOT minimalist — data-rich and immersive.

**Color System:**
- Background (deepest): #0F0F1A
- Card surface: #1A1A2E
- Divider/elevated: #252540
- Brand primary (buttons, active tabs, highlights): #7C3AED (electric violet)
- Secondary accent (labels, secondary info): #06B6D4 (cyan blue)
- Bullish/Buy: #22C55E (green — Taiwan market convention)
- Bearish/Sell: #EF4444 (red)
- Level colors: Bronze #CD7F32 / Silver #C0C0C0 / Gold #FFD700 / Platinum #E5E4E2 / Diamond #B9F2FF
- Text primary: #F8F8FF / Text secondary: #94A3B8

**Typography:** Google Fonts — use a modern mono font for prices and P&L numbers (e.g. JetBrains Mono or Roboto Mono), sans-serif for UI labels (e.g. Nunito or Inter).

**Layout:** Mobile-first (375px width). 8dp spacing grid. Bottom navigation bar with 5 tabs: Trading, Positions, Journal, Stats, Ranking.

**Design all 7 screens:**

1. **Auth Screen** — dark card on deep background, app logo with violet glow, email/password fields with subtle border, primary CTA "登入" button in violet, Google sign-in and anonymous login as secondary options, switch between login/register.

2. **Trading Screen (main)** — top: instrument selector (FITX 大台 / MFTX 小台) with current price in large mono font + green/red delta; middle: embedded order form with BUY (full green) / SELL (full red) large buttons, quantity +/- control, price input, estimated margin display; bottom: mini position summary card. Dense but breathable.

3. **Positions Screen** — list of open positions as cards showing: instrument, direction badge (BUY/SELL), entry price, current price, unrealized P&L (large, colored), open time; swipe-right-to-close action. Empty state with illustration.

4. **Journal Screen** — trade history list with date grouping, each row: direction badge + instrument + P&L + reflection score icon; tap to expand detail page showing full trade info + 5-minute reflection Q&A answers + tags. Filter chips at top (date / instrument / tag).

5. **Stats Screen** — P&L equity curve (fl_chart line chart, violet fill under curve), key metrics row: total P&L / win rate / max drawdown / avg R:R displayed as data cards; time filter tabs (1W / 1M / 3M / All); MDD warning banner if threshold exceeded.

6. **Ranking Screen** — current user rank card at top with level badge (Bronze→Diamond) and XP bar; global leaderboard list with avatar, nickname, level badge, P&L score, rank number; achievements grid below (locked/unlocked states with glow effect on unlocked). Animated rank-up celebration effect.

7. **Reflection Flow (modal/overlay)** — post-trade 5-minute countdown timer (circular progress, violet), 4 structured questions with radio/text inputs, submit button. Timer ticking creates urgency.

**Component style:** Cards with subtle 1px violet-tinted borders (#7C3AED at 20% opacity). Numbers always in monospace font. Buy/Sell always as filled colored badges, never text-only. Level badges as gradient chips. Active bottom nav tab with violet indicator dot + icon color change.

**Do NOT include:** light theme variants, Material You dynamic color, BottomSheet for order entry, any real-money transaction UI.
```
