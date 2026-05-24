# MVP1 UI Redesign Spec

> 2026-05-06。現有 OrbitBar、Spotlight、Create Panel 全部重做。
> 原則：shadcn/ui 內建 + 成熟 package，不硬刻。RWD 優先。產品級品質。

---

## 一、技術棧

```
React 18 + TypeScript + Vite
shadcn/ui v4 (Base UI 版)
Tailwind CSS v4
motion (framer-motion) — 動畫
```

---

## 二、設計原則（來自 pal 和 sonnet-search 的 UX 建議）

### 交易 Dashboard UX 避雷

1. **不要 hover-only 操作** — 觸控裝置沒 hover，所有操作要有可見按鈕
2. **不要小於 44px 的觸擊目標** — WCAG + Apple HIG 標準
3. **不要硬刻 modal/portal** — 用 shadcn Dialog/Sheet，自帶 focus trap + scroll + a11y
4. **不要單字母/符號代替文字** — 用完整 label，L0 用戶看不懂抽象符號
5. **不要遮住主要內容** — 面板/filter 要可收合或半透明
6. **狀態持久化** — filter、選中的 basket 跨 session 保留
7. **空狀態要有引導** — 每個面板都要有 empty state + CTA
8. **主焦點是圖表** — 其他 UI 都是輔助，不能搶注意力

### 產品級 Component API 設計

- **Controlled components** — value + onChange，狀態抬升到 store
- **Accessibility** — aria-selected、aria-label、keyboard navigation、focus-visible
- **Semantic HTML** — 正確的 heading 層級、role
- **Error/Loading states** — 每個 async 操作有 skeleton + error fallback
- **Motion** — 用 `motion` layoutId 做 tab 切換指示器動畫，`prefers-reduced-motion` respect

---

## 三、組件重做方案

### A. Basket 切換（取代 OrbitBar）

**shadcn 組件：Tabs + ScrollArea**

```
Desktop (1024+):
┌──────────────────────────────────────────────────────────────┐
│ 📊 US Sectors │ 📈 Index │ 🏛 Macro │ 🌍 Global │ ... │ [+] │
│ ═══════════                                                  │
│ (active underline, motion layoutId)        (← scroll →)      │
└──────────────────────────────────────────────────────────────┘

Mobile (<768):
┌─────────────────────────────┐
│ 📊 US Sectors  ▼            │  ← dropdown / bottom sheet
└─────────────────────────────┘
```

**行為：**
- 每個 tab 顯示 emoji + 完整名稱
- `TabsList` + `overflow-x-auto whitespace-nowrap` 處理溢出
- Active tab：底部 underline indicator，用 `motion` layoutId 做平滑動畫
- 最右側固定 [+] 按鈕（sticky position）
- Custom basket 用不同色調（primary/20 背景）區分
- 刪除 < > 箭頭

**RWD 策略：**
- Desktop：水平 tab bar，左右滾動
- Tablet：同上，tab 名稱自動縮短（只顯示 emoji + 前 6 字）
- Mobile：收合為 dropdown select，或點擊展開 bottom sheet
- Touch target：48px 高度

**Package：** `npx shadcn add tabs scroll-area` + `motion`

**參考：**
- TradingView watchlist tabs — 水平滾動 + 清晰 label
- TradingView minimalistic display mode: https://www.tradingview.com/blog/en/minimalistic-display-mode-for-watchlist-41721/
- BuildUI animated tabs recipe: https://buildui.com/recipes/animated-tabs

---

### B. Basket 瀏覽（取代 Spotlight）

**shadcn 組件：Sheet (desktop: side="right", mobile: side="bottom") + ScrollArea + Card**

```
┌──────────────────────────────────────────┐
│  Baskets                       [×]       │
│  [🔍 Search baskets...]                  │
│──────────────────────────────────────────│
│  PRESET                                  │
│  ┌─ 📊 US Sectors ─────────────────────┐│
│  │ SPY | XLK XLF XLV XLY XLP...  11隻 ││
│  └─────────────────────────────────────┘│
│  ┌─ 📈 US Index Internal ─────────────┐│
│  │ SPY | SPY QQQ IWM DIA          4隻 ││
│  └─────────────────────────────────────┘│
│  ...                                     │
│                                          │
│  MY BASKETS                              │
│  ┌─ 🔥 Quantum+Nuclear ── custom ─────┐│
│  │ SPY | IONQ OKLO SMR RGTI NNE   5隻 ││
│  │              [Edit] [Delete]        ││
│  └─────────────────────────────────────┘│
│                                          │
│  (empty state: "還沒有自訂清單，點 + 建立")  │
│                                          │
│  [+ New Basket]                          │
└──────────────────────────────────────────┘
```

**行為：**
- 用 shadcn `Sheet`，不用 createPortal 硬刻
- 內部用 `ScrollArea` 確保滾動正確（解決現在 bug）
- Preset 和 Custom 分 section（有標題分隔）
- 點擊 card → 切換 basket → 自動關閉
- Custom card hover/focus 顯示 Edit/Delete 操作
- 有搜尋框（filter by name）
- Empty state 有引導文案

**RWD 策略：**
- Desktop：Sheet side="right"，寬 400px
- Mobile：Sheet side="bottom"，max-h-[70vh]，有 handle 可拖

**Package：** `npx shadcn add sheet scroll-area card`

**參考：**
- TradingView watchlist widget docs: https://www.tradingview.com/widget-docs/widgets/watchlists/
- Trading Watchlist Dashboard template: https://nagaredesignstudio.com/dashboard-and-cms/trading-watchlist-dashboard-template/

---

### C. Symbol 多值輸入（Create Panel）

**shadcn 組件：Combobox (multiple + chips) — shadcn v4 Base UI 版已內建！**

```
SYMBOLS (5/1000)
┌──────────────────────────────────────────────────┐
│ IONQ × │ OKLO × │ SMR × │ RGTI × │ NNE ×       │
│ [搜尋或貼上代碼...]                                │
│ ┌──────────────────────────┐                     │
│ │ AAPL — Apple Inc.        │ ← autocomplete      │
│ │ AMZN — Amazon.com        │                     │
│ │ AMD — Advanced Micro...  │                     │
│ └──────────────────────────┘                     │
└──────────────────────────────────────────────────┘
```

**核心發現：shadcn v4 有原生 `Combobox` multiple + chips！**

```tsx
// shadcn v4 內建 — 不需要 emblor 或其他外部套件
<Combobox items={suggestions} multiple value={tickers} onValueChange={setTickers}>
  <ComboboxChips>
    <ComboboxValue>
      {tickers.map(t => <ComboboxChip key={t}>{t}</ComboboxChip>)}
    </ComboboxValue>
    <ComboboxChipsInput placeholder="搜尋或貼上代碼..." />
  </ComboboxChips>
  <ComboboxContent>
    <ComboboxEmpty>找不到標的</ComboboxEmpty>
    <ComboboxList>
      {(item) => <ComboboxItem key={item} value={item}>{item}</ComboboxItem>}
    </ComboboxList>
  </ComboboxContent>
</Combobox>
```

**加上 paste handler（~15 行）：**
- 監聽 `onPaste` event
- 解析 clipboard：split by `/[,\s\n]+/`，trim，toUpperCase
- 去重，過濾空值
- 不認識的 ticker → toast "XXX not found, skipped"
- 支援：`IONQ, OKLO, SMR` / `IONQ\nOKLO\nSMR` / `IONQ OKLO SMR`

**RWD：**
- 所有尺寸都用同一個 Combobox，chips 自動換行（flex-wrap）
- Dropdown 在 mobile 自動調整位置

**Package：** `npx shadcn add combobox`（shadcn v4 Base UI 版內建）

---

### D. Filter Panel（新組件）

**shadcn 組件：ToggleGroup type="multiple"**

```
Desktop — OrbitMap 左上角覆蓋層：
┌─────────────────────────────────────────────────────┐
│ SECTOR  [全部] [量子] [核能] [太空] [半導體] [AI]     │
│ COLOR   [全部] [●紅] [●橙] [●黃] [●綠] [●藍] [...]  │
│                                      [重置全部]      │
└─────────────────────────────────────────────────────┘

Mobile — 底部可收合 overlay：
┌─────────────────────────────────────────────────────┐
│ [Filter ▲]                                          │
│ SECTOR  [全部] [量子] [核能] [太空] ...              │
│ COLOR   [全部] [●紅] [●橙] ...                      │
│                                      [重置全部]      │
└─────────────────────────────────────────────────────┘
```

**行為：**
- 用 shadcn `ToggleGroup type="multiple"`
- 每個 ToggleGroupItem：
  - Sector pill：中文 label（from tag_translations API）
  - Color pill：色點 + 名稱（e.g. `●紅`）
- 點擊 toggle 開/關（AND 邏輯）
- 「全部」toggle 重置該行
- 「重置全部」清除所有 filter
- 狀態持久化到 `basket_tag_states` API（切 basket 恢復）
- 不符合的 symbol：OrbitMap 消失、Table 置灰排底

**RWD 策略：**
- Desktop：固定在 OrbitMap 左上角，半透明背景
- Tablet：同上但 pill 更緊湊
- Mobile：收合為 filter icon，點擊展開 bottom sheet

**Package：** `npx shadcn add toggle-group`

**參考：**
- Sharpely RRG zone filter: https://sharpely.in/relative-rotation-graph
- Sharpely zone filter demo: https://www.youtube.com/watch?v=mJLIah4cIbk
- OptionStrat flow filter: https://optionstrat.com/flow
- Linear filter chips

---

## 四、RWD 整體策略

### Breakpoints

| 範圍 | 佈局 | Basket 切換 | Filter | Basket 瀏覽 |
|------|------|------------|--------|------------|
| Mobile (<768) | 單欄：圖表→表格 | Dropdown / bottom sheet | 收合 icon → bottom sheet | Sheet bottom |
| Tablet (768-1024) | 圖表佔 2/3 + 表格 | 水平 tabs（名稱縮短） | 覆蓋層（緊湊） | Sheet right |
| Desktop (1024+) | 圖表 + 側邊表格 | 水平 tabs（完整名稱） | 覆蓋層 | Sheet right |

### 共通規則
- Touch target ≥ 44px（Apple HIG + WCAG）
- 字體最小 12px，正文 14px
- 所有互動元素有 hover + focus-visible 狀態
- `prefers-reduced-motion` 時關閉動畫
- Dark mode first（交易者偏好）

### 參考產品的 Mobile 處理方式
- **TradingView Mobile**：底部 tab 導航（4-5 項）+ 上方 watchlist dropdown
- **Robinhood Mobile**：搜尋主導，首頁即 watchlist
- **Bloomberg Mobile**：精簡版，核心功能優先

---

## 五、Package 清單

| 需求 | Package | 安裝 |
|------|---------|------|
| Basket tabs | shadcn Tabs | `npx shadcn add tabs` |
| 滾動溢出 | shadcn ScrollArea | `npx shadcn add scroll-area` |
| Basket 瀏覽面板 | shadcn Sheet | `npx shadcn add sheet` |
| 卡片 | shadcn Card | `npx shadcn add card` |
| Symbol 輸入 | shadcn Combobox (v4 multi+chips) | `npx shadcn add combobox` |
| Filter pills | shadcn ToggleGroup | `npx shadcn add toggle-group` |
| Tab 動畫 | motion | `npm i motion` |
| Toast | shadcn Toast (已有) | — |

---

## 六、設計參考 URL

### 交易 Dashboard UI
- TradingView watchlist widget: https://www.tradingview.com/widget-docs/widgets/watchlists/
- TradingView minimalistic mode: https://www.tradingview.com/blog/en/minimalistic-display-mode-for-watchlist-41721/
- Trading Dashboard template: https://nagaredesignstudio.com/dashboard-and-cms/trading-watchlist-dashboard-template/
- OptionStrat flow filter: https://optionstrat.com/flow
- OptionStrat flow tutorial: https://optionstrat.com/tutorials/unusual-options-flow

### RRG / Filter
- Sharpely RRG zone filter: https://sharpely.in/relative-rotation-graph
- Sharpely zone filter video: https://www.youtube.com/watch?v=mJLIah4cIbk

### shadcn/ui 參考
- shadcn blocks gallery: https://ui.shadcn.com/blocks
- shadcn examples: https://ui.shadcn.com/examples
- Animated tabs recipe: https://buildui.com/recipes/animated-tabs

### Mobile RWD
- TradingView mobile watchlist: https://www.youtube.com/watch?v=uYhs2AHXpQQ
- TradingView mobile vs desktop: https://www.kotakneo.com/hi/stockshaala/trading-view/mobile-vs-desktop-key-differences/

---

## 七、實施順序

```
1. 安裝所有 shadcn 組件 + motion
2. Basket Tabs（取代 OrbitBar）→ prototype → 用戶 review
3. Symbol Combobox Input（修 Create Panel）→ prototype → 用戶 review
4. Basket Browser Sheet（取代 Spotlight）→ prototype → 用戶 review
5. Filter Panel ToggleGroup（新功能）→ prototype → 用戶 review
```

**每個組件的開發流程：**
1. 安裝 shadcn 組件（`npx shadcn add xxx`）
2. 寫組件（<200 行限制）
3. 本地驗證 + Chrome 截圖
4. 給用戶 review → 修改 → 確認
5. 整合進主 App
