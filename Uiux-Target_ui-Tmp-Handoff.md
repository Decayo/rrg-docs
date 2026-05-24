# filter該怎麼設計？


## pplx 的討論

這是一個很有深度的產品設計問題，讓我從三個維度拆解：**Filter UX 設計**、**AI-assisted Filter Builder** 以及 **「自建 ETF」作為 RRG 標的** 的架構。

***

## Filter 設計的參考標竿

你已用過的三個工具各有不同哲學：

| 工具 | Filter 模式 | 強項 | 弱點 |
|---|---|---|---|
| **OptionStrat Flow** | 規則組合（條件式）+ 顏色高亮  [optionstrat](https://optionstrat.com/flow) | 顏色 tag 幫助視覺辨識 | 需要用戶自己懂參數意義 |
| **Unusual Whales** | 多維度 checkbox + 快速預設 | 預設 template 降低門檻 | filter 組合爆炸沒有引導 |
| **ChadderFlow** | 情境化 darkpool/flow 情感分類 | 語意化標籤設計直覺 | 彈性較低 |
| **Sharpely RRG** | Zone filter（quadrant 直接點擊）  [youtube](https://www.youtube.com/watch?v=mJLIah4cIbk) | 一鍵 quadrant 篩選極快 | 只有象限，無法跨維度 |

**最值得參考的額外案例**：
- **Notion filter builder** — 用 AND/OR 可視化邏輯鏈，非技術用戶也能組合複雜條件，每個 pill 都可點擊編輯
- **Airtable filter** — 漸進式揭露（先選欄位 → 再選 operator → 再填值），完全避免空結果
- **Linear issue filter** — 輸入框 + 快速 suggest chip，一個 input 統一入口

***

## AI Chatbot 互動式 Filter 建構

這個方向非常有潛力，設計要點：

**對話流程設計**（減少摩擦的核心）

```
用戶：我想找 Leading 象限、只看我有持倉的
AI：已為你建立 filter：象限=Leading + 持倉標的=開啟
    要加入 momentum > 某個門檻嗎？[加入] [這樣就好]
```

- **Intent → Filter 的映射**：AI 理解「最近強勢的」= RS-Momentum 高 + Leading 象限；「進場機會」= Improving 象限
- **Upload 分析**：用戶上傳持倉截圖/CSV → AI 自動解析 ticker list → 幫你建立持倉 watchlist filter，這正好能接下面的 Custom ETF 邏輯
- **Filter 建好後可儲存 + 命名**（如 OptionStrat 的 saved filter ），AI 可以幫你取個有意義的名字 [optionstrat](https://optionstrat.com/blog/introducing-enhanced-options-flow)

**技術上**：一個輕量 function-calling 架構就夠，AI 輸出結構化 filter JSON，前端直接套用。

***

## 「自建 ETF」作為 RRG 標的

這是你這個 idea 裡**最有差異化**的部分，市面上幾乎沒有產品做到。

**概念架構**：

```
用戶持倉 (positions + weights)
        ↓
  加權組合計算每日 NAV
        ↓
  合成一條 price series（synthetic ticker）
        ↓
  作為 RRG 的一個節點 vs benchmark（SPY/QQQ）
```

- **實作方式**：每個交易日計算持倉加權報酬，生成一條 synthetic index，直接對 benchmark 算 JdK RS-Ratio & RS-Momentum
- **衍生應用**：
  - 多個基金/帳戶各自變成一個節點，**做帳戶間 RRG 比較**（哪個部位組合動能最強）
  - 朋友/追蹤的 KOL 公開持倉也能當一個節點
  - 「我的多頭組合 vs 我的空頭組合」雙節點 RRG

**資料來源**：可以串接 FUGLE/Firstrade API 拿即時持倉，或讓用戶手動填 / CSV 上傳。

***

## 整體架構建議

整合起來，你的系統可以分三層：

1. **Ingestion 層** — 持倉上傳（CSV / API / AI 解析截圖）→ synthetic ticker 計算
2. **Filter Engine** — 可存規則的 JSON schema，AI chatbot 作為 natural language → filter rule 的轉換器
3. **RRG View** — 標準象限圖 + 支援 synthetic ticker 節點 + Zone filter（一鍵 quadrant 篩選 ） [youtube](https://www.youtube.com/watch?v=mJLIah4cIbk)

目前業界最接近你這個 vision 的是 **Portfolio Visualizer** 的 backtest 持倉合成功能，但他們沒有 RRG，也沒有 AI filter builder — 你的整合是真正的空白市場。


## 另外考量到易用性
- 是否接下來的排版也需要對手機部分進行微調細化？ 
- 目前來說 排版基本上都是靠著硬尻？ 我剛看了一下並沒有做 RWD



### pplx rwd guide


# Project Frontend Guidelines

Claude Code 必讀規範。每次生成前端 code 都必須遵守此文件。

---

## 1. RWD 響應式設計（MANDATORY）

**所有前端元件都必須支援 RWD。** 不得生成只適配桌機的版面。

### Viewport Meta Tag（每個 HTML 必須有）
```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

### Breakpoints（統一標準，不得自創其他數值）

| 名稱 | 最小寬度 | 代表裝置 |
|------|----------|----------|
| `phone` | 0px（default） | iPhone SE, Android |
| `tablet` | 768px | iPad portrait |
| `laptop` | 1024px | iPad landscape, 小筆電 |
| `desktop` | 1280px | 一般桌機 |
| `wide` | 1440px | 寬螢幕 |

### 寫法規則：Mobile-First（先寫手機，往上疊）

```css
/* ✅ 正確：mobile-first */
.component {
  padding: 1rem;          /* 手機 default */
}
@media (min-width: 768px) {
  .component { padding: 2rem; }   /* tablet+ */
}
@media (min-width: 1280px) {
  .component { padding: 3rem; }   /* desktop+ */
}

/* ❌ 錯誤：不要用 max-width 當主要 breakpoint */
@media (max-width: 767px) { ... }
```

---

## 2. Layout Pattern by Device

### Dashboard / App Layout

```css
.app-layout {
  display: grid;
  height: 100dvh;
  overflow: hidden;
  grid-template-columns: 1fr;           /* phone: 無 sidebar */
  grid-template-rows: 56px 1fr;
}
@media (min-width: 768px) {
  .app-layout { grid-template-columns: 64px 1fr; }   /* tablet: icon sidebar */
}
@media (min-width: 1280px) {
  .app-layout { grid-template-columns: 240px 1fr; }  /* desktop: full sidebar */
}
```

**規則：**
- 手機：Sidebar 隱藏，改用 bottom tab bar 或 hamburger
- Tablet：Sidebar 收合為 icon only（64px）
- Desktop：Sidebar 展開含文字 label（240px）
- **只能有一個 scroll region**：`.main-content { overflow-y: auto }`，`body { overflow: hidden }`

### Chart Grid

```css
.chart-grid {
  display: grid;
  gap: 1rem;
  grid-template-columns: 1fr;                              /* phone: 單欄 */
}
@media (min-width: 768px) {
  .chart-grid { grid-template-columns: repeat(2, 1fr); }  /* tablet: 兩欄 */
}
@media (min-width: 1280px) {
  .chart-grid { grid-template-columns: repeat(auto-fill, minmax(400px, 1fr)); } /* desktop: 彈性 */
}
```

### Table / Data List

手機版 table 必須轉成 **card view**：

```css
@media (max-width: 767px) {
  .data-table thead { display: none; }
  .data-table tr {
    display: block;
    padding: 1rem;
    border-bottom: 1px solid var(--color-border);
  }
  .data-table td {
    display: flex;
    justify-content: space-between;
  }
  .data-table td::before {
    content: attr(data-label);
    color: var(--color-text-muted);
    font-size: 0.75rem;
    font-weight: 500;
  }
}
```

HTML table 對應寫法：
```html
<td data-label="股票">AAPL</td>
<td data-label="損益">+$234</td>
```

---

## 3. Design Tokens（統一變數，不得 hardcode 數值）

```css
:root {
  /* Spacing — 4px base */
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-6: 1.5rem;    /* 24px */
  --space-8: 2rem;      /* 32px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */

  /* Typography */
  --text-xs:   clamp(0.75rem,  0.7rem  + 0.25vw, 0.875rem);
  --text-sm:   clamp(0.875rem, 0.8rem  + 0.35vw, 1rem);
  --text-base: clamp(1rem,     0.95rem + 0.25vw, 1.125rem);
  --text-lg:   clamp(1.125rem, 1rem    + 0.75vw, 1.5rem);
  --text-xl:   clamp(1.5rem,   1.2rem  + 1.25vw, 2.25rem);

  /* Touch target */
  --touch-target: 44px;   /* 所有可點擊元素最小尺寸 */

  /* Transitions */
  --transition: 180ms cubic-bezier(0.16, 1, 0.3, 1);

  /* Radius */
  --radius-sm: 0.375rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;
}
```

---

## 4. Typography 規則

- **Body 最小 16px**，手機上低於 16px 會觸發 iOS 自動 zoom
- **Buttons / Nav：14px**（`--text-sm`）
- **Labels / Badges：12px**（`--text-xs`），這是絕對下限
- **Heading 用 display font**，Body 用 body font，混用要明確
- 每頁最多 4-5 種字體樣式組合

---

## 5. 觸控友善規則

```css
/* 所有互動元素都要滿足 44px touch target */
button, a, [role="button"], input, select {
  min-height: var(--touch-target);    /* 44px */
  min-width: var(--touch-target);
}

/* Mobile 不能用 hover-only 互動 */
@media (hover: none) {
  .tooltip:hover { display: none; }           /* 停用 hover tooltip */
  .tooltip:focus-within { display: block; }   /* 改用 focus */
}
```

---

## 6. Dark Mode（每個頁面都要支援）

```css
:root, [data-theme="light"] {
  --color-bg: #f7f6f2;
  --color-surface: #f9f8f5;
  --color-text: #28251d;
  --color-text-muted: #7a7974;
  --color-border: #d4d1ca;
  --color-primary: #01696f;
}

[data-theme="dark"] {
  --color-bg: #171614;
  --color-surface: #1c1b19;
  --color-text: #cdccca;
  --color-text-muted: #797876;
  --color-border: #393836;
  --color-primary: #4f98a3;
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme]) {
    --color-bg: #171614;
    --color-surface: #1c1b19;
    --color-text: #cdccca;
    --color-text-muted: #797876;
    --color-border: #393836;
    --color-primary: #4f98a3;
  }
}
```

---

## 7. Accessibility 基本要求

- 每個 `<img>` 必須有 `alt` 屬性
- Icon-only button 必須有 `aria-label`
- Heading 層級不能跳（h1 → h2 → h3，不能 h1 直接到 h3）
- 每頁只有一個 `<h1>`
- 所有 form input 必須有關聯的 `<label>`
- Focus ring 不能移除（`:focus-visible` 要保留）

---

## 8. Performance 規則

```html
<!-- 圖片必須加這些屬性 -->
<img src="..." alt="..." width="800" height="450" loading="lazy" decoding="async">

<!-- JS 要 defer -->
<script src="app.js" defer></script>

<!-- Font 要 preconnect -->
<link rel="preconnect" href="https://fonts.googleapis.com">
```

---

## 9. 禁止事項

不得出現以下（AI 常犯的錯誤）：

- ❌ `position: fixed` 疊加多個（sticky header + sticky footer + banner = 手機沒內容可見）
- ❌ 所有文字 `text-align: center`（只有 hero 標題可以 center）
- ❌ 每個元件都用相同大的 `border-radius`（要有層級）
- ❌ `border-left: 3px solid <color>` 當 card 裝飾
- ❌ Icon 放在有色圓形背景裡當 feature section 裝飾
- ❌ 不用 `localStorage` 或 `sessionStorage`（sandbox 環境會 crash，用 in-memory variable）
- ❌ Hardcode pixel 值（用 token 變數）
- ❌ 只寫桌機版不寫手機版

---

## 10. 驗收 Checklist

每次完成一個元件或頁面，自我檢查：

- [ ] `viewport` meta tag 存在
- [ ] 手機版（375px）單欄佈局正確
- [ ] Tablet（768px）版面合理
- [ ] Desktop（1280px）版面完整
- [ ] Touch target >= 44px
- [ ] Body 文字 >= 16px
- [ ] Dark mode 切換正常
- [ ] 沒有 hardcode px 值（用 token）
- [ ] Table 在手機上變 card view
- [ ] 只有一個 scroll region


## 其他問題
- 目前登入畫面也需要美化一下
- 但美化應該放後面點 我認為討論ux 使用者體驗和功能掀起來優先
- 但RWD這種介於UI和UX之間的東西 先確保功能上沒問題 比方說觸控,不遮擋 
  - 細項如gap之類的priority可以先不用