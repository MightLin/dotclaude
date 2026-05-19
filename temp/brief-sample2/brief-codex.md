# Design Brief: futures-trading-system-ui
產生時間：2026-05-16
Mode：greenfield

## 資訊足夠性檢查
- 已知資訊: 產品是台灣期貨紙上交易與交易日誌 App，目標使用者為嚴肅零售交易者；平台為 Flutter iOS / Android / Web，Mobile 375px 優先；核心模組包含 Auth、Trading、Historical Playback、Reflection、Analytics、Ranking；既有設計基準要求深色主題、台灣市場紅跌綠漲語義、BottomNavigationBar、8dp 間距、FL Chart 圖表、直接在交易頁下單。
- 缺少的必要資訊: 無會阻止 brief 產出的必要資訊。精準品牌主色尚未指定。
- 可安全假設: 以深色專業交易介面為主，主色可先採低飽和金融藍灰 / 墨黑基底，交易語義色遵守台灣市場：漲/買為綠，跌/賣為紅；Web 版作為響應式延伸，不先做獨立桌面資訊架構。
- 高風險假設: 是否需要支援券商級多螢幕高密度交易桌面版、是否要加入社群動態流、是否有特定品牌識別色或 logo 系統。
- 是否可產出 brief: YES

## 產品概覽
- 目的: 建立一套給台灣期貨交易者使用的紙上交易、歷史回放、交易紀錄、反思與績效成長系統 UI，讓使用者能在低風險環境練習交易決策並累積可檢討的交易資料。
- 目標使用者: 嚴肅零售交易者，特別是想練習台指期 / 小台策略、追蹤交易紀律、透過統計與反思改善績效的人。
- 平台: Flutter 跨平台，優先 mobile app（375px 基準），Web 作為響應式 bonus。

## 功能範圍
- 主要功能 / 新增功能: Auth 登入、即時行情與下單、部位管理、歷史行情回放、交易日誌、交易後 5 分鐘反思、績效統計、排行榜與成就成長。
- 核心流程: 使用者登入或匿名進入；選擇商品 TXF / MXF；在交易頁觀察即時報價與關鍵數據；直接輸入市價 / 限價單並送出；在持倉頁管理部位與保證金；成交或平倉後自動觸發 5 分鐘反思；交易紀錄進入日誌；統計頁呈現 P&L、回撤與勝率；排行榜頁顯示等級、成就與全域排名。
- 主要畫面: 登入 / 註冊、主交易頁、持倉頁、歷史回放頁、交易日誌頁、反思問卷頁、統計分析頁、排行榜 / 成就頁、設定 / 個人資料頁、錯誤與空狀態畫面。

## 視覺方向
- 調性: 專業、冷靜、長時間可讀；介面應像嚴肅交易工作台，而不是行銷頁或遊戲大廳。排行榜與成就可以有克制的成長感，但不能干擾交易判讀。
- 色彩偏好: 深色基底；漲 / 買 = 綠色，跌 / 賣 = 紅色，遵守台灣市場慣例。品牌主色暫定為低飽和金融藍灰或青藍作為互動強調色，避免大面積紫色、亮色主題、過度漸層。
- 互動狀態需求（loading / empty / error）: 需要設計 loading、empty、error、offline / reconnecting、order pending、order rejected、trade filled、reflection due、no positions、no journal records、no ranking data 等狀態。
- 黑名單（不要的元件或模式）: 不使用亮色主題；不做卡片堆疊式行銷首頁；交易下單不使用 BottomSheet 或獨立頁面；不使用過度裝飾背景、漸層球、玻璃擬態；不要讓重要行情數字被插圖或文案干擾。

## 技術限制
- UI 套件 / 框架: Flutter；狀態管理使用 flutter_bloc + equatable；圖表使用 fl_chart；字體使用 google_fonts；本地儲存使用 Hive；Firebase Auth / Firestore / Cloud Functions；Dio + web_socket_channel 串接市場資料。
- 不可違反的專案規則: Clean Architecture 分層：UI -> Bloc -> UseCase -> Repository -> Datasource；Widget 或 Bloc 不直接呼叫 Firestore / 富果 API；domain 層純 Dart；不使用 Provider、Riverpod、GetX；不使用 SharedPreferences、sqflite、http、Retrofit；不 commit .env 或 API 金鑰；ThemeData 統一管理樣式，不用 inline style。

## 既有設計脈絡（feature-extension / design-guide-refresh）
- 既有 design-guide 摘要: 深色主題；台灣交易語義色為漲 / 買綠、跌 / 賣紅；底層導航包含交易、持倉、日誌、統計、排名；間距使用 8dp 倍數；圖表使用 FL Chart；載入用 CircularProgressIndicator；輕量錯誤用 SnackBar，需要確認的錯誤用 Dialog。
- 既有頁面 / 截圖 / 程式碼觀察: 本 brief 依專案文件從零規劃，尚未檢查現有畫面截圖或可執行 localhost。若後續進入實作，應再讀取 lib/presentation/pages/ 與共用 widgets，避免與既有程式結構衝突。
- architecture / business-logic / tech-stack 摘要: 架構定義 Auth、Trading、Historical Playback、Reflection、Analytics、Ranking、FugleAPI、TaifexData、FirebaseRemote 等模組；資料流為使用者操作 -> Page -> Bloc -> UseCase -> Repository 介面 -> Datasource；business-logic.md 目前無內容；技術棧以 Flutter + Firebase + Firestore + Fugle WebSocket + TAIFEX CSV 為主。

---

## 給 AI 設計工具的 Prompt
請為一個「台灣期貨紙上交易 + 交易日誌 App」從零設計完整 UI。產品服務嚴肅零售交易者，核心價值是即時模擬交易、歷史行情回放、交易後 5 分鐘反思機制，以及透過統計與遊戲化成長改善交易紀律。平台是 Flutter iOS / Android / Web，請優先設計 mobile 375px 體驗，並讓版面可自然延伸到 Web。

整體視覺必須是深色主題，適合交易者長時間盯盤使用。風格要專業、冷靜、高資訊密度但不壓迫，像可日常使用的交易工作台，不像行銷 landing page。請使用台灣市場慣例：漲 / 買使用綠色，跌 / 賣使用紅色。品牌主色可採低飽和金融藍灰或青藍作為互動強調色，但避免亮色主題、大面積紫色、過度漸層、玻璃擬態與純裝飾背景。所有數字、損益、保證金、部位與風險資訊都要可快速掃描。

請設計完整資訊架構與主要畫面：
1. 登入 / 註冊 / 匿名進入：支援 Email、Google、匿名體驗。
2. 主交易頁：顯示 TXF / MXF 商品切換、即時報價、漲跌、成交價、關鍵行情資訊、帳戶虛擬餘額、保證金狀態、下單表單。下單表單必須直接在交易頁內操作，不使用 BottomSheet 或跳轉獨立頁。支援市價 / 限價、買 / 賣、口數、價格、預估保證金、送單確認與送單結果。
3. 持倉頁：顯示目前部位、均價、未實現損益、保證金占用、可用餘額、平倉操作與風險提示。
4. 歷史回放頁：顯示 TAIFEX 歷史資料回放、時間軸、播放 / 暫停、速度控制、目前回放時間、回放中的行情與下單入口。
5. 日誌頁：列出交易紀錄，可依日期、商品、方向、盈虧、是否已反思篩選。每筆紀錄要顯示進出場、口數、損益、持倉時間與反思狀態。
6. 反思問卷頁：在交易後 5 分鐘自動提示，提供結構化問題，例如進場理由、出場理由、是否遵守計畫、情緒狀態、可改善事項。介面要快速填寫，不要像長篇表單。
7. 統計頁：用 fl_chart 可實作的圖表概念呈現 P&L 曲線、最大回撤、勝率、平均盈虧、交易頻率、商品分布、紀律分數。
8. 排名 / 成就頁：顯示等級、全域排行榜、成就徽章與成長進度，但風格要克制，不能破壞專業交易調性。
9. 設定 / 個人資料頁：帳戶、通知、資料同步、風險偏好、初始虛擬資金設定。

主導航請使用底層導航，核心 tab 為「交易、持倉、日誌、統計、排名」。歷史回放可放在交易頁的模式切換或次級入口，也可以在資訊架構中提出更好的位置，但必須容易從交易工作流進入。間距使用 8dp 倍數，元件樣式應可透過 Flutter ThemeData 統一管理。請避免巢狀卡片與過度浮動卡片；交易頁可以使用密度較高的面板布局，但必須保持清楚分區。

請同時設計 loading、empty、error、offline / reconnecting、order pending、order rejected、trade filled、reflection due、no positions、no journal records、no ranking data 等狀態。錯誤提示分輕重：輕量錯誤適合 SnackBar，需要確認的風險或送單錯誤適合 Dialog。請確保 mobile 小螢幕上文字不重疊、不截斷重要數字，按鈕大小可觸控，行情數字與下單操作有明確優先順序。

技術限制：Flutter、flutter_bloc、fl_chart、google_fonts、Hive、Firebase Auth / Firestore / Cloud Functions、Dio、web_socket_channel。不可設計需要 Provider、Riverpod、GetX、Syncfusion、SharedPreferences、sqflite、http 或 Retrofit 才能合理實作的模式。UI 只描述 presentation 層，不假設 Widget 會直接呼叫 Firestore 或富果 API。
