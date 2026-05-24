# OrbitFlow Feature Backlog

> 功能待辦清單。🔲 待做 | 🟡 需深度討論 | ✅ 完成
> MVP 標記：`[M1]` MVP1 | `[M2]` MVP2 | `[M3]` MVP3

### M1 執行順序（2026-05-20 Grill 確定）

```
E11 Note CRUD + 渲染（flat+tag, per-symbol UI）
  ↓
E11 TD Sequential + GMMA pullback compute
  ↓  ← 合併
U45 Web Worker（EMA/RSI/KDJ/MACD + DeMark + GMMA batch）
  ↓
U46 Mini Chart Grid（復用信號渲染層 + shared viewport）
  ↓
剩餘 M1（E1 Auth / E7 部署 / E10 Guest）
```

### 文件關係

| 文件 | 角色 |
|------|------|
| **本文件 (doc/13)** | 全量 backlog，所有功能的單一來源 |
| `docs/24` | MVP1 實施計劃（W1-W6），開發以此為準 |
| `docs/23/rrg-tag-basket-spec.md` | Tag 系統架構參考（schema + namespace） |
| `docs/23/orbitflow-product-positioning.md` | 產品定位 + User Story + 定價方案 |
| `docs/22` | 歷史參考（Basket MVP1 舊 spec），被 doc/24 取代 |

---

## 一、CDD 補債（技術債）

| # | 項目 | 類型 | 狀態 |
|---|------|------|------|
| T1 | DevPlayground 補 LoginPage / UserMenu / SettingsSheet | 前端 | ✅ |
| T2 | 新組件 Chrome 截圖視覺驗證 | 前端 | ✅ DOM 驗證通過 |
| T3 | 穩定組件加 `@sealed` 註解 | 前端 | ✅ TailSlider/AuthGuard/UserMenu |

---

## 二、前端功能（近期）

### 2A. Symbol Table + DetailPanel

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| F1 | **TREND 欄改迷你 K 線** | 需討論：table 只有 close 數據，真 K 線需 OHLCV | 🟡 [M2] |
| F2 | **展開 icon 開 DetailPanel** | 每行 ⤢ icon，click → onSelect → DetailPanel | ✅ |
| F3 | **主標的 Timeframe tag** | Badge: `YF · D` / `IBKR · W`，UserMenu 左側 | ✅ |
| F4 | **DetailPanel 入口統一** | 雙擊 row / 展開 icon / click OrbitMap 圓點 → handleSymbolClick | ✅ |

### 2B. UI/UX 修正

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| U1 | **UserMenu 可見性** | 加 bg-accent/50 + border，修 button 嵌套 | ✅ |
| U2 | **Loading skeleton** | shadcn Skeleton 替換所有 Loading 文字（4 處） | ✅ |
| U3 | **RWD 基礎** | Mobile-first breakpoints, touch target 44px, table→card view | 🔲 [M2] |
| U4 | **Login 頁面美化** | 乾淨的登入/註冊表單，品牌 logo | 🔲 [M1] |
| U5 | **效能優化** | bundle 分析 + 500 node 效能檢查 | 🔲 [M1] |
| U6 | **iOS Safari 適配** | visibilitychange cleanup, canvas 上限, passive events, GPU memory | 🔲 [M2] |
| U7 | **Brush/RRG 日期對齊** | ✅ 重構完成：date-based windowing，見 docs/17 | ✅ |
| U8 | **Brush 左右 handle 可拖拉 resize** | D3 handle 目前隱藏(useBrush:54)，需顯示+自由 resize | 🔲 [M2] |
| U9 | **W3 Focus Layout regression — basket panel 入口** | AppToolbar ▼ 按鈕觸發 BasketBrowserPopover | ✅ |
| U10 | **W3 Focus Layout regression — table reopen** | table 關閉後左下角 chip 可重開 | ✅ |
| U11 | **W3 Focus Layout regression — loading/error 狀態** | RRG hero 上 loading/error overlay | ✅ |
| U12 | **DetailPanel bottom bar** | timeframe + opacity 移到底部 bar，header 精簡 | ✅ |
| U13 | **DetailPanel click toggle** | 點 header = 展開/收合，拖 header = 拖曳（移動 >4px 區分） | ✅ |
| U14 | **DetailPanel click outside collapse** | 點 RRG canvas 收合，點 toolbar/table/brush 不收合 | ✅ |
| U15 | **RRG dot → DetailPanel force expand** | openRevision 確保 dot 點擊永遠展開 | ✅ |
| U16 | **Benchmark sparkline** | Table benchmark row 顯示 trend sparkline | ✅ |
| U17 | **Table header tooltips** | shadcn Tooltip 替代 native title（%D/%W/%R/AMP/PRICE/QUAD） | ✅ |
| U18 | **Table action tooltips** | ⤢ 和 TV icon hover 說明 | ✅ |
| U19 | **Table focused row** | 點 RRG dot → table row 高亮 + 自動滾動到中間 | ✅ |
| U20 | **Catmull-Rom spline trails** | RRG 軌跡線改 bezierCurveTo（tension 0.5），更平滑 | ✅ |
| U21 | **上限 1000** | 前後端 MAX_SYMBOLS 200 → 1000 | ✅ |
| U22 | **Custom scrollbar** | 4px 寬半透明 thumb（spotlight-scroll class） | ✅ |
| U23 | **Layout B: Workstation** | ResizablePanelGroup 左 RRG + 右 chart+table，拖曳分隔線 | ✅ |
| U24 | **DockedChart** | 右側固定 K 線，預設 benchmark，點 dot 切換，點空白回 benchmark | ✅ |
| U25 | **Symbol 顏色 golden angle** | `symbolColor()` hash → HSL 均勻分佈，排序不影響顏色 | ✅ |
| U26 | **K chart 狀態繼承** | 切換標的保留 zoom/pan offset（logical range persist） | ✅ |
| U27 | **BasketDrawer 修正** | shadcn Sheet → createPortal（解決 CSS specificity 問題） | ✅ |
| U28 | **FilterPanel 預設展開** | `expanded = true`，全寬 strip | ✅ |
| U29 | **RRG 長方形填滿** | 移除強制正方形，填滿左側 panel | ✅ |
| U30 | **Toolbar prototype 對齊** | ◎ brand-mark + nav tabs + search bar + gradient avatar + layout/bell icons | ✅ |
| U31 | **Popover 三欄修正** | Recommended(presets) / Recent(empty) / My baskets(custom)。刪除 hardcoded 假資料 | ✅ |
| U32 | **RRG hover dim 動畫** | hover dot → 其他 symbols 漸變淡出（lerp 0.3, ~80ms）| ✅ |
| U33 | **RRG click glow 呼吸** | selected trail + dot 3 層亮色 glow（symbolGlow 82% lightness），sine 呼吸脈動 | ✅ |
| U34 | **品牌統一** | OrbitFlow / OrbitMap，brand.ts + index.html title | ✅ |
| U35 | **Login v3 redesign** | SIGN IN / CREATE ACCOUNT tabs，Mac titlebar，glassmorphism | ✅ |
| U36 | **自助註冊** | register endpoint public，前端 register() 接入 | ✅ |
| U37 | **AI 持倉截圖解析** | Grok via PAL MCP 解析持倉截圖 → JSON {symbols, benchmark, name} | ✅ 驗證通過 |
| U38 | **AI chatbot 美化** | 對話 UI 太陽春，Phase 2 需大改（markdown 渲染、流式更好的 UX） | 🔲 [M2] |
| U39 | **K chart intraday 時間顯示** | 4h 以下 (4h/1h/30m/15m) x-axis 顯示 `M/D HH:mm`（UTC seconds → browser local），daily 保留 lightweight-charts 自動格式 | ✅ |
| U40 | **K chart timeline 三色 tag** | 藍=viewport 左右邊界、橘=RRG window；vertical line 跨全 chart 高度，bottom label 碰撞時往上堆疊（TAG_W=80px）。灰色 mouse 由 lightweight-charts native crosshair 提供 | ✅ |
| U41 | **K chart 預設 viewport = RRG window ± 5 bars padding** | 取代硬編 DEFAULT_BARS。Range 優先序：saved zoom > RRG window ± 5 > fitContent。saved-zoom 切 interval 時繼承 | ✅ |
| U42 | **K chart dynamic chunk load** | Intraday (15m) IBKR 一次只給 30 天。用 30 天 tile 分段 fetch：初始 1 tile → viewport 滾到 50% 時背景預抓下一個 tile → 最多 24 tiles (2 年)。需評估架構可行性（OHLCV endpoint 加 endDate param、前端 useOHLCVTiles hook 合併 tiles） | 🟡 [M2] |
| U43 | **K chart pre-load / background render** | 切 symbol 前背景預載 OHLCV（類似 RRG prewarm）。需討論：何時觸發？cache 策略？記憶體上限？目前架構 DockedChart 在 symbol 切換時重建 chart instance，pre-load 需要分離 data fetch 和 chart render | 🟡 [M2] |
| U44 | **K chart instance reuse（核心效能）** | chart create 一次（deps=[]），切換只做 setData。DockedChart 移除條件渲染，chart 不再 unmount。��含 viewport 拆分（data/viewport/markers 各自 effect） | ✅ |
| U45 | **指標 + 信號 Web Worker** | EMA/RSI/KDJ/MACD + DeMark Sequential + GMMA pullback 全部進 Worker。Batch precompute per-timeframe。Transferable ArrayBuffer 零拷貝。合併 E11.5 信號計算 | 🔲 [M1] 依賴 U44 |
| U46 | **多股 mini chart grid** | responsive 欄數（fit 螢幕）。顯示：K+Vol+GMMA+PriceNote+ColorTag。展開 btn→DockedChart、TV btn→TradingView（帶 interval+studies）。拖曳任一 chart 全部同步平移（shared viewport）。Timeframe 鎖 Day/Week。排序雙向同步 Table + 手動拖曳 reorder（basket 級 position 持久化）。點擊=高亮，展開=切 DockedChart | 🔲 [M1] 依賴 U44+U45+E11 |
| U47 | **RSI/KDJ/MACD pane title + 參數設定** | ✅ 標題顯示（series title: RSI(14), K/D/J, MACD/DIF/DEA）。🔲 點擊調參數���Phase 2） | ✅ 部分 |
| U48 | **交叉信號三角 tag** | KDJ K/D + MACD DIF/DEA 交叉 markers。detectCrossovers() + useChartCrossMarkers hook。顏色跟 bull/bear setting | ✅ |
| U49 | **漲跌配色全局設定（台灣/美國模式）** | CSS variable `--color-bull`/`--color-bear` + Settings 切換 + localStorage。影響 K bar/text/%/volume/sparkline/crossover markers | ✅ |
| U50 | **盤前/盤後/夜盤延長時段顯示** | Backend: `useRTH=False`（ibkr.py line 135），每 bar 加 session tag（pre/rth/post/night）。Frontend: 三 toggle 按鈕（盤前☀️/盤後🌅/夜盤🌙）只在 <1D 出現。預設 fetch all 但只 render RTH；toggle 開 → filter + setData（U44 reuse）。Glow: pre=黃, post=橘, night=紫（用背景色帶 HistogramSeries）。Fetch 策略：一次 useRTH=False 全拿，前端 session filter 控制顯示。Daily RRG 不受影響；intraday RRG 未來需決策是否包含 extended bars | 🟡 [M2] |
| U51 | **期貨合約支持（ES/NQ Globex）** | IBKR contract: `Future("ES", "GLOBEX", "USD")` + `ContFuture` 連續合約。需 Globex market data subscription。23 小時交易 → `useRTH=False` 拿完整 session。UI: 合約選擇器 + 到期月切換。依賴 U50 延長時段基礎 | 🟡 [M2] 依賴 U50 |
| U52 | **Pinned Basket Dock（釘選快切）** | 浮動左側 vertical list。Emoji collapsed + hover expand + click switch + drag reorder + right-click delete。localStorage order persist | ✅ |
| U53 | **Chatbot Mode 頁面** | 獨立頁面（非 workstation overlay）。AI 對話全寬主畫面，OrbitMap 作為對話中的附件/快照。用戶可丟 RRG 截圖讓 AI 分析。適合 L0「幫我看市場」場景。Workstation Mode 的 AI 維持懸浮視窗形式（快速建 basket + 一句話問答）。兩個 mode 共享 function call 底層（createBasket / setFilter / switchTimeframe） | 🔲 [M2] |

### 2D. 上線前大功能

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| D1 | **15min 數據來源 (IBKR)** | Full stack interval 參數化（8 files）。15m/1h/1d/1wk。IBKR barSizeSetting + yfinance interval | ✅ |
| D2 | **Cache TTL 分 interval** | 15m→3min, 1h→5min, 1d/1wk→10min | ✅ |
| D3 | **多股並列預覽（Finviz style）** | 2 欄網格滾動，K 線 + 量 + 價格。Table 不變，換 detail panel。📷 `image copy 28.png` | 🔲 [M1] |
| D4 | **時間軸移到上方** | K 線時間刻度移到頂部。📷 `image copy 27.png` 綠圈 | 🔲 [M1] |
| D5 | **Detail header 完善** | +$5.50 實際漲跌 + 修 %W/%R 遺失。📷 `image copy 27.png` 紅圈 | ✅ |
| D7 | **Per-timeframe chart range** | 切 timeframe 各自獨立 range，切 symbol 繼承。_savedRanges Map | ✅ |
| D8 | **股票全名顯示** | hover RRG tooltip + DockedChart header + detail Info tab | ✅ |
| D11 | **Detail tabs (Info/Holdings/DP)** | chart 下方 resizable detail 區。Info: 全名+sector+mcap placeholder。Holdings: ETF 成分 placeholder。DP: Dark Pool placeholder。📷 `image copy 30.png` | ✅ placeholder |
| D12 | **Detail Info 填充** | yfinance info API 或 IBKR contract details。sector/exchange/mcap/52w/盤前盤後 | 🔲 [M2] |
| D13 | **Detail Holdings 填充** | ETF top 10 成分股+權重。用現有 etf_routes | 🔲 [M2] |
| D14 | **Header 中英文名** | symbols_us.json 加 zh-TW 欄位。或用 tag_translations | 🔲 [M2] |
| D9 | **Baseline timeframe 架構** | RRG interval selector + link 雙向 + IBKR 15m/1wk 驗證通過 | ✅ |
| D10 | **KDJ-based RRG 算法** | 現有 JdK 太滯後，加 KDJ 變體。method 參數切換 | 🔲 [M1] |
| D6 | **Chatbot UI 大改** | markdown、多輪、streaming、自訂 message types | 🔲 [M2] |

#### Baseline Timeframe 設計規則

```
RRG baseline = 全局 timeframe（影響 RRG trail + sparkline + %D/%W/%R + brush）
Detail chart = 獨立 timeframe（只影響 K 線 candles + RSI/KDJ/MACD）

免費：RRG baseline 鎖 1D（收盤更新）
付費：RRG baseline 可切 15M/1H/1D/1W，3min 自動更新
高級：即時 streaming

時區：以用戶瀏覽器時區為主
```

#### D1 研究結果：IBKR TWS API 15min 數據

**方案：TWS API + `ib_async` + IB Gateway（headless）**

```
架構：IB Gateway (localhost:4001) → FastAPI MarketDataService → Redis cache → WebSocket → React
```

| 項目 | 詳情 |
|------|------|
| API call | `reqHistoricalData(barSizeSetting="15 mins", keepUpToDate=True)` |
| 即時更新 | `keepUpToDate=True` → `historicalDataUpdate` callback 推送未完成 bar |
| Python 客戶端 | `ib_async`（ib_insync 的維護版），async/await 友好 |
| Rate limit | 15min bars 無硬性限制（已放寬），但避免 15s 內重複請求 |
| 同時請求 | 最多 50 個 simultaneous historical requests |
| 數據延遲 | 有 Level 1 訂閱 = 即時；無訂閱 = `reqMarketDataType(3)` 15min 延遲 |
| Gateway | IB Gateway headless，比 TWS Desktop 輕量，適合 server |
| 更新策略 | startup 拉最近 N 天 → `keepUpToDate=True` 持續推送 → 每 3-5 min 推到前端 |
| 錯誤處理 | error 162/165/200 → auto-retry + backoff |
| 未來擴展 | symbols 多時用 Redis cache + batch rotation |

### 2C. 核心導航（W4）

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| N1 | **Spotlight ⌘K** | Center sheet 三欄 browser（Recent/Recommended/My baskets）+ 搜尋 | ✅ |
| N2 | **Basket Browser ▼** | B1 Anchored popover（toolbar 下拉，三欄，淡 backdrop） | ✅ |
| N3 | **Basket Create Drawer** | 右側 Sheet（Manual/AI/Import tabs），symbol multi-paste | ✅ |
| N4 | **Symbol multi-paste** | useSymbolPaste hook（comma/space/tab 分割 + 去重 + 大寫） | ✅ |
| N5 | **Emoji picker row** | 44px 預覽 + 12 快選 emoji | ✅ |
| N6 | **⌘N 快捷鍵** | 開啟 Basket Drawer | ✅ |
| N7 | **Spotlight ↔ Drawer 聯動** | Spotlight 內 Create new → 關 Spotlight 開 Drawer | ✅ |

---

## 三、EPIC

### E1. 用戶管理系統 (Auth + Profile + Onboarding)

**MVP1 scope:** email/password + Google OAuth + token refresh。見 doc/24 W1.4。
**目前狀態：** Dev bypass 可用，Google OAuth Client ID 已申請（.env），SQLite user model 有了。

| 子項 | 說明 | 狀態 |
|------|------|------|
| E1.1 | Google OAuth 真實接入 | 🔲 [M1] |
| E1.2 | Profile 頁面（顯示 name/email/plan tier） | 🔲 [M1] |
| E1.3 | Onboarding（首次登入預設 basket 自動載入） | 🔲 [M1] |
| E1.4 | Token refresh（靜默 refresh 或 re-login） | 🔲 [M1] |
| E1.5 | 帳號刪除 / 資料匯出（GDPR） | 🔲 [M2] |

---

### E2. Basket / Tag / Filter 系統

> **2026-05-05 架構決策：** flat basket 被 tag-based 架構取代。
> 6 namespace：`sector`, `color`, `cap`, `beta`, `signal`, `custom`。
> MVP1 只露出 `sector` + `color`（Phase 1 原則）。
> 架構詳見 `docs/23/rrg-tag-basket-spec.md`。實施順序見 `docs/24` W1-W3。

**目前狀態：** PG 遷移完成。Tag 系統後端就緒。W4 導航完成。W5 Filter 進行中。

#### M1 核心

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| E2.1 | Basket CRUD | Create + Edit + Delete + Auto-focus + Toast | ✅ |
| E2.2 | Basket 雲端同步 | PostgreSQL + basket_symbols 關聯表 | ✅ |
| E2.3 | **Tag schema 建表** | 8 張表（PG）含 tags/translations/symbol_tags/basket_tag_states | ✅ |
| E2.4 | **Tag CRUD API** | 8 endpoints（含 symbol-map 批次） | ✅ |
| E2.5 | **System tag seed** | 18 sector + 16 color + zh-TW/en translations | ✅ |
| E2.6 | **symbol_tags 初始綁定** | 57 symbol-tag bindings（quantum/nuclear/semi/ai/ev/crypto/defense） | ✅ |
| E2.7 | **Filter Panel UI** | TAG FILTER bar，展開/收合，sector pills | ✅ |
| E2.8 | **Filter 行為：OrbitMap** | tag filter → mergedHidden → RRG canvas 隱藏 | ✅ |
| E2.9 | **Filter 行為：Table** | filteredOut → opacity-20 + sort 排底 | ✅ |
| E2.10 | **basket_tag_states 持久化** | 前端 debounce 600ms 同步。Custom: API PUT。Preset: localStorage | ✅ |
| E2.11 | **Create Panel：tag group 搜尋** | 搜尋「量子」→ `[TAG] sector:quantum (18 隻)` → 一鍵加入 | 🔲 |
| E2.12 | **Symbol 貼上支援** | useSymbolPaste hook（comma/space/tab/newline 解析） | ✅ |
| E2.13 | **Benchmark autocomplete** | HTML datalist 12 個常用 ETF/Index 自動完成 | ✅ |
| E2.14 | **OrbitBar pill 放大** | 44px (w-11 h-11)，text-base emoji | ✅ |
| E2.15 | **Color tag：dot ring color** | OrbitMap 圓點外圈 2px 彩色環（useColorTags hook） | ✅ |
| E2.16 | **Color tag：Table badge** | symbol 名後方 6px 色塊 | ✅ |
| E2.17 | **Color tag：Quick-tag modal** | Table badge click → 16 色 grid popover，assign/remove via API | ✅ |
| E2.18 | **上限 1000** | 前後端移除 200 限制 | 🔲 |
| E2.19 | **AI mock endpoint** | `POST /api/ai/chat` → structured reply + action payloads | ✅ |

#### M2 延伸

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| E2.20 | createBasket AI tool（with `tags_to_apply`） | AI 解析 intent → 建 basket + 掛 tag | 🔲 |
| E2.21 | Plan tier gate | 免費 1 個自訂 + 預設，付費無限 | 🔲 |
| E2.22 | Quick add symbol | 從 table/OrbitMap 右鍵加入 basket | 🔲 |
| E2.23 | Basket 分享 | 分享 basket 給其他用戶 | 🔲 |
| E2.24 | Spotlight 搜尋 | filter title/symbol/benchmark + tag 語法 | 🔲 |
| E2.25 | Spotlight 多入口 | title 點擊、雙擊 pill、▼、空格鍵 | 🔲 |
| E2.26 | cap/beta tag 前端露出 | Filter Panel 擴充到 4 行 | 🔲 |
| E2.27 | Tag 管理 UI | 新增/刪除自訂 tag | 🔲 |
| E2.28 | Mute mode | 灰化保留 vs 消失的切換 | 🔲 |
| E2.29 | OrbitBar 動畫 | 切換/新增 basket 時 pill 滑動 | 🔲 |
| E2.30 | 統一 Custom 和 + 入口 | 右上角 Custom 按鈕與 [+] 行為統一 | 🔲 |
| E2.31 | Create Panel 效能 | 打字/操作卡頓排查 | 🔲 |
| E2.32 | Emoji Picker | emoji-mart 套件 | 🔲 |

#### M3 願景

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| E2.33 | Pseudo symbol / 合成 ETF | 用戶持倉加權 → synthetic price → OrbitMap 節點 | 🔲 |
| E2.34 | Portfolio upload | CSV / API / AI 截圖 → ticker list | 🔲 |
| E2.35 | signal/custom namespace 前端 | 技術指標 tag + 使用者自訂分組 | 🔲 |

**UX 設計參考：** Notion filter builder、Airtable 漸進揭露、Linear chip input、Sharpely quadrant click
**三層架構：** Ingestion（持倉上傳）→ Filter Engine（AI + tag 規則）→ OrbitMap View。詳見 `docs/target_ui/tmp-handoff.md`。
**測試策略：** Tag CRUD + Filter 行為 + basket_tag_states 持久化 + 1000 node 效能

---

### ~~E3. Color Tag 系統~~ → 已合併至 E2

Color Tag 現為 tag 系統的一個 namespace (`color`)。
所有相關子項已移入 E2.15-E2.17。
架構詳見 `docs/23/rrg-tag-basket-spec.md` 第四節。

---

### E4. 定價 + 付費系統

**排入：MVP2。** 等差異化功能成形再定價。定價方案參考 `docs/23/orbitflow-product-positioning.md` 第十節。
**目前狀態：** 完全未做。plan_tier 欄位已在 user model。

| 子項 | 說明 | 狀態 |
|------|------|------|
| E4.1 | 定價頁設計 | 🔲 [M2] |
| E4.2 | Stripe/Paddle Checkout 整合 | 🔲 [M2] |
| E4.3 | 付費 gate UI | 🔲 [M2] |
| E4.4 | 訂閱管理 | 🔲 [M2] |
| E4.5 | Trial 機制 | 🔲 [M2] |

---

### E5. Dark Pool + 進階指標

**排入：MVP3。** 依賴 E6 (Notes) 基礎和差異化功能需求確認。
**目前狀態：** DetailPanel 有 "Dark Pool Levels — coming soon" placeholder。

| 子項 | 說明 | 狀態 |
|------|------|------|
| E5.1 | DP 數據源 | 🔲 [M3] |
| E5.2 | DP Level 渲染 | 🔲 [M3] |
| E5.3 | KDJ-on-RS 指標 | 🔲 [M3] |
| E5.4 | 型態辨識 | 🔲 [M3] |
| E5.5 | 指標自訂 | 🔲 [M3] |

---

### E6. Symbol 資訊 + Notes 系統

**排入：MVP2。** notes 表在 MVP1 W1 預建但不做 UI。
**目前狀態：** DetailPanel 只顯示 ticker，沒有全名、info、notes。

| 子項 | 說明 | 狀態 |
|------|------|------|
| E6.1 | DetailPanel 全名顯示 | 🔲 [M2] |
| E6.2 | Table 全名（hover/展開） | 🔲 [M2] |
| E6.3 | Info icon（市值、sector、交易所） | 🔲 [M2] |
| E6.4 | Notes 系統（per-symbol markdown） | 🔲 [M2] |
| E6.5 | Notes DB schema（notes + note_tags + tsvector） | 🔲 [M2] |
| E6.6 | AI Notes | 🔲 [M3] |
| E6.7 | i18n 全名（symbol_info + translations） | 🔲 [M2] |
| E6.8 | Graph DB（Neo4j/AGE） | 🔲 [M3] |

---

### E7. 部署 + 基礎建設

**MVP1 scope:** PostgreSQL 遷移 + Cloudflare Tunnel + 域名。見 doc/24 W5。
**目前狀態：** Docker Compose 骨架就緒。決策：Arch self-host + Cloudflare Tunnel。
**成本：** $0（self-host + pg_dump backup）+ 域名年費

| 子項 | 說明 | 狀態 |
|------|------|------|
| E7.1 | SQLite → PostgreSQL | Docker Compose + postgres:16-alpine + asyncpg | 🔲 [M1] |
| E7.2 | Arch self-host 部署 | Docker Compose + Cloudflare Tunnel 反向代理 | 🔲 [M1] |
| E7.3 | 域名 + HTTPS | Cloudflare DNS + Tunnel，域名 orbitflow.xxx | 🔲 [M1] |
| E7.4 | CI/CD pipeline | GitHub Actions → build → test → deploy | 🔲 [M2] |
| E7.5 | 監控 | Uptime、Sentry、usage analytics | 🔲 [M2] |
| E7.6 | Redis cache | API response cache + session | 🔲 [M2] |
| E7.7 | 品牌統一 | OrbitFlow / OrbitMap，頁面 title + favicon + logo | 🔲 [M1] |

---

### E8. AI 日報 / 週報系統

**排入：MVP2-M3。** 核心差異化功能。依賴 basket URL routing + 截圖。

| 子項 | 說明 | 狀態 |
|------|------|------|
| E8.1 | **Basket URL routing** | 每個 basket 有唯一 URL（`/b/{id}`），支持分享 + AI 截圖入口 | 🔲 [M2] |
| E8.2 | **AI 每日截圖** | 排程截圖 basket RRG + K chart → 存 S3/local → 附到報告 | 🔲 [M2] |
| E8.3 | **AI 日報生成** | AI 分析 basket 變化 → markdown 報告（sector rotation、quadrant 變動、異常 symbol） | 🔲 [M2] |
| E8.4 | **AI 週報生成** | 週級摘要 + 趨勢比較 + 持倉建議 | 🔲 [M2] |
| E8.5 | **Plan tier gate** | Free: 0 reports。Pro: 3 baskets 日/週報。Ultimate: 20 baskets | 🔲 [M2] |
| E8.6 | **Basket symbol 上限** | 每 basket 最多 20 symbols。超出 → AI 建議壓縮/分組 | 🔲 [M2] |
| E8.7 | **截圖 → AI 回饋循環** | 截圖給 AI 分析 → AI 產出 insight → 附在報告中 | 🔲 [M3] |

### E9. Composite Symbol（自建 ETF / Group-as-Node）

**排入：MVP1-M2。** 核心差異化。Basket 合成為單一 RRG 節點 + tag group 展開規則。

**設計原則：**
- UI 不變（用戶照常輸入 symbols）— 邏輯在 resolver + compute 層
- AI 是主要 expression 產生者（chatbot function call）
- 優先：custom basket → composite mode（已有 CRUD）
- 次優先：color/sector tag group → composite node

**Expression 語法（NAI-style，AI 產出為主）：**
```
composite:my_port::{weighted}::0.5>NVDA::0.3>TSLA::0.2>AMD
sector:quantum::{equal}           → 均值聚合為一個節點
sector:quantum::{mcap_top10}      → 前 10 大市值加權
```

**Context-dependent resolution：**

| 輸入 | RRG (OrbitMap) | Table |
|------|---------------|-------|
| `sector:quantum` | 一個合成節點（聚合） | 展開前 10 大市值個股 |
| `composite:my_port` | 一個合成節點 | 展開成分股 + 權重 |
| `NVDA` | 一個點 | 一列 |

**Data volume 問題 🟡：** N composites × M symbols = 大量 price fetch。需 cache 策略。

#### M1 核心

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| E9.1 | **Basket composite mode** | baskets 加 `composite_mode` + `basket_symbols.weight`。toggle 顯示為一個節點 | 🔲 |
| E9.2 | **Synthetic price compute** | 後端：symbol list + weights → synthetic OHLCV series | 🔲 |
| E9.3 | **RRG composite node** | synthetic OHLCV → RS-Ratio/Momentum → OrbitMap 一個 dot | 🔲 |
| E9.4 | **Expression parser** | 解析 `{weighted}::0.5>NVDA::0.3>TSLA` 語法 | 🔲 |
| E9.5 | **Cache 策略** | composite price precompute + TTL（Redis / LRU / batch） | 🟡 |
| E9.6 | **AI function call** | `createComposite()` / `setWeights()` chatbot tools | 🔲 |

#### M2 延伸

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| E9.7 | **Tag group composite** | sector/color tag → 自動聚合為 RRG 節點 | 🔲 |
| E9.8 | **Market cap data** | mcap-weighted 聚合（需 mcap 資料源） | 🔲 |
| E9.9 | **嵌套 composite** | composite 包含另一個 composite → 遞迴計算 | 🔲 |
| E9.10 | **持倉匯入** | AI 截圖 → 自動建立加權 composite | 🔲 |
| E9.11 | **多帳戶比較** | 不同 composite 在同一 RRG 比較 | 🔲 |
| E9.12 | **KOL 公開持倉** | 追蹤 KOL composite node | 🔲 |

**架構決策 🟡：**
1. Cache：Redis vs in-memory LRU vs precompute cron？
2. Data fetch 爆量：10 composites × 50 symbols = 500 series — batch？
3. 聚合時機：request-time vs background precompute？
4. Sector proxy：用 ETF（XLK）還是自己算？

### E10. Guest / Anonymous 使用者

**排入：MVP1-M2。** 進站不強制登入，全功能可用。

| 子項 | 說明 | 狀態 |
|------|------|------|
| E10.1 | **Anonymous session** | 進站自動建 localStorage session，不建 DB user | 🔲 [M1] |
| E10.2 | **功能不限** | Guest 可用所有功能（RRG、K chart、basket、filter） | 🔲 [M1] |
| E10.3 | **註冊 merge** | 註冊時 localStorage data → DB user（baskets、tags、viewport state） | 🔲 [M2] |
| E10.4 | **URL 分享** | Guest 可透過 URL 看別人的 basket（read-only） | 🔲 [M2] |

### E11. Note 系統 + Computed Signals（視覺信號層）

**排入：MVP1-M2。** 核心 UX 差異化。RRG tail 上直接看信號，K chart 同步渲染。

**2026-05-20 Grill 決策：**
- **Note = Obsidian-like flat entity + tag**，不用 type enum。tag 決定 note 是什麼（`#signal:support`, `#signal:dp_level`, `#source:manual`, `#source:ai`）
- **Note 和 Computed Signal 是兩套系統**，不混合。Note = 用戶/AI 手動建立。Signal = 算法產出。
- **Note schema nullable symbol** — 允許全局筆記，但 M1 UI 只露出 per-symbol（DetailPanel signal tab）
- **M1 手動建立入口**：DetailPanel signal tab "+" 按鈕（非 chart 上右鍵）
- **AI createNote**：M1 只做 console log 驗證 Grok function call schema 正確，不做 UI
- **Computed signals per-timeframe**：跟 TradingView 一樣，每個 timeframe 各自計算
- **Batch precompute** + 合併進 U45 Web Worker（DeMark/GMMA/EMA/RSI/KDJ/MACD 一次設計）
- **VPVR 不做**，留給 TradingView 跳轉
- **TradingView 跳轉帶 interval + studies 參數**

#### Data Model: Note（flat + tag）

```
notes {
  id, user_id,
  symbol?,        -- nullable（全局筆記 or per-symbol）
  price?,         -- nullable（有 price = K chart 水平線，沒有 = 純文字）
  content,        -- markdown
  created_at, updated_at
}

note_tags → 複用現有 tag 系統 (tags + note_tags 關聯表)
```

#### TD Sequential 算法（參考 `docs/ref_pine/CC-yata.pine`）

```
Setup (1-9):  buy = close < close[4] 連續計數 | sell = close > close[4]
Countdown:    Standard buy = close < low[2] | sell = close > high[2]
              Aggressive buy = low < low[2]  | sell = high > high[2]
M1 scope:     Standard 9/13 + Aggressive a9/a13（不做 Stealth、不做 S/R 線）
```

#### 子項

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| E11.1 | **Note CRUD API** | `POST/PUT/DELETE /api/notes`。flat + tag，nullable symbol | 🔲 [M1] |
| E11.2 | **Note K chart 渲染** | 有 price 的 note → `createPriceLine()` 水平線 + label | 🔲 [M1] |
| E11.3 | **Note RRG tail 渲染** | tail 節點附近小 icon 表示該 symbol 有 note | 🔲 [M1] |
| E11.4 | **Note DetailPanel UI** | signal tab "+" 按鈕建立，列表顯示 per-symbol notes | 🔲 [M1] |
| E11.5 | **TD Sequential compute** | Standard 9/13 + Aggressive a9/a13。per-timeframe batch。進 Web Worker | 🔲 [M1] |
| E11.6 | **TD Sequential K chart markers** | bar 上方/下方數字標記（buy=綠, sell=紅，漸變透明度） | 🔲 [M1] |
| E11.7 | **TD Sequential RRG tail 渲染** | tail 末端三角形 + 數字 | 🔲 [M1] |
| E11.8 | **GMMA 交叉回踩 icon** | pullback 信號。K chart + RRG tail 同步顯示 | 🔲 [M1] |
| E11.9 | **Global hide toggle** | 一鍵隱藏/顯示所有 tail 標籤和 note（Settings or toolbar toggle） | 🔲 [M1] |
| E11.10 | **AI function call: createNote** | M1 只驗證 Grok function call schema（console log），不做 UI | 🔲 [M1] |
| E11.11 | **AI function call: getSignals** | AI 讀取某 symbol 的所有信號 + notes → 分析 | 🔲 [M2] |
| E11.12 | **Per-symbol signal summary** | DetailPanel Signals tab 列出所有 notes + 技術信號 | 🔲 [M2] |
| E11.13 | **全局 note UI** | 不綁 symbol 的筆記入口，等 M2 Chatbot Mode 頁面 | 🔲 [M2] |

---

### E12. AI Workstation Control（Chatbot → UI 操控）

**排入：MVP1-M2。** 殺手級應用。自然語言控制 workstation UI。
**目前狀態：** BasketDrawerAITab 只能解析截圖建 basket，需要大幅升級。

**架構：** Chatbot floating panel（workstation mode） → function call dispatch → UI state update

#### Function Call 清單

| Function | 說明 | 範例 |
|----------|------|------|
| `createTmpBasket` | Grok+Polygon 找標的 → 建臨時 basket → 顯示在 RRG | 「有什麼 AI 股？」 |
| `switchBasket` | 切換到指定 basket | 「切到核能」 |
| `setFilter` | 設定 tag filter | 「只看半導體」 |
| `createPriceNote` | 建立 price note | 「NVDA 135 有大 DP」 |
| `getScreenshot` | 取得當前 RRG + K chart 截圖 → AI 分析 | 「現在板塊怎樣？」 |
| `highlightSymbol` | 高亮某 symbol | 「NVDA 在哪？」 |
| `setInterval` | 切換 timeframe | 「切到 15 分鐘」 |
| `zoomToRange` | 設定 chart viewport | 「看最近一個月」 |

#### 子項

| # | 項目 | 說明 | 狀態 |
|---|------|------|------|
| E12.1 | **Chatbot floating panel 升級** | 從 BasketDrawer AI Tab 獨立出來。Floating panel（右下角），markdown 渲染，streaming | 🔲 [M1] |
| E12.2 | **Function call dispatcher** | AI response 含 `actions[]` → dispatch 到對應 handler → UI state 更新 | 🔲 [M1] |
| E12.3 | **createTmpBasket** | Grok+Polygon 查標的 → 建臨時 basket（不存 DB）→ 直接渲染 RRG。用戶可選「保留」或「丟棄」 | 🔲 [M1] |
| E12.4 | **getScreenshot** | `html2canvas` 或 `canvas.toDataURL()` 截取 RRG + K chart → base64 → 送 AI 分析 | 🔲 [M1] |
| E12.5 | **createPriceNote via chat** | 自然語言 → 解析 symbol + price + type → 建立 price note（依賴 E11.2） | 🔲 [M1] |
| E12.6 | **UI 控制 functions** | switchBasket / setFilter / highlightSymbol / setInterval / zoomToRange | 🔲 [M1] |
| E12.7 | **對話上下文** | 多輪對話保持 context（當前 basket、symbol、filter 狀態）讓 AI 知道現在看什麼 | 🔲 [M2] |
| E12.8 | **快捷指令** | `/create`、`/filter`、`/screenshot` 等 slash command 快捷鍵 | 🔲 [M2] |

---

## 四、UX 流程（User Behavior Map）

### 4A. 認證流程
```
未登入 → AuthGuard → /login
  ├─ [Google Sign-In] → JWT → /
  ├─ [Email/Password] → JWT → /
  └─ [Skip Login (Dev)] → JWT → /
→ App 載入 → useAuth 驗證 token
```

### 4B. 主操作流程
```
App 主頁
  ├─ OrbitBar: preset pills / custom pills / [+] / [▼]
  ├─ Filter Panel (OrbitMap 左上角):
  │   ├─ Sector pill → toggle filter (AND)
  │   ├─ Color pill → toggle filter
  │   ├─ 「全部」→ reset row
  │   └─ 「重置全部」→ all tags on
  ├─ Benchmark Line: Brush 拖曳 → 時間窗口
  ├─ OrbitMap: Hover=Tooltip, Click=DetailPanel
  ├─ Symbol Table:
  │   ├─ Click → toggle visibility
  │   ├─ Double-click → DetailPanel
  │   ├─ 展開 icon → DetailPanel
  │   ├─ Color badge → Quick-tag modal
  │   └─ TV → TradingView 新分頁
  └─ DetailPanel (浮動, 可拖曳/縮放):
      K 線 + GMMA + RSI/KDJ/MACD + OrbitMap markers
```

### 4C. Settings 流程
```
UserMenu → Settings → SettingsSheet (右側滑出)
  ├─ Data: Source / Timeframe
  ├─ Display: Label mode
  ├─ Language: zh-TW / en
  └─ Appearance: Theme
→ 自動存 API + localStorage
```

### 4D. Filter Panel 狀態流程
```
Filter Panel → 點 sector pill → basket_tag_states 更新
  ├─ OrbitMap：不符合的 symbol 消失
  ├─ Table：不符合的列置灰、排底、不參與 sort
  └─ 切換 basket → 載入該 basket 的 basket_tag_states → 恢復 filter
```
