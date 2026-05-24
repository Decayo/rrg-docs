# RRG 實作文件

> ⚠️ **本檔嚴重過時（待重寫）**
> - React 18 → 實際 React 19
> - Plotly RRGChart → 已砍，改 Canvas RRGCanvas
> - 檔案結構 → 已大幅變動（hooks 28 / components 51）
> - API 路徑 → 已加 /api/tags /api/notes /api/ai 等
> - Phase B-E 未來路線 → 多項已完成
>
> 列為 R-rewrite 議題（Session 7b/c 處理）。**參考時請對照當前 code 驗證**。
> 當前架構鳥瞰可看：[INDEX.md](INDEX.md) + [31-refactor-backlog.md](31-refactor-backlog.md)

## 架構總覽

```
┌─────────────────────────────────────────────────┐
│                   Frontend                       │
│  React 18 + TypeScript + Vite                    │
│                                                  │
│  ┌──────────┐ ┌──────────┐ ┌─────────────────┐  │
│  │ TopBar   │ │ Controls │ │ BasketSelector  │  │
│  └──────────┘ └──────────┘ └─────────────────┘  │
│  ┌──────────────────────────────────────────┐    │
│  │ BenchmarkLine (D3 SVG + brushX)         │    │
│  │ - 折線圖 + draggable time window        │    │
│  │ - 角落圓點 + 日期 + 中間 %/days          │    │
│  └──────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────┐    │
│  │ RRGChart (Plotly)                        │    │
│  │ - 四象限散點圖 + trail lines             │    │
│  │ - Hover popup: symbol info + sparkline   │    │
│  │ - 未來: DP marker shapes (▲◆★)          │    │
│  └──────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────┐    │
│  │ SymbolTable                              │    │
│  │ - SYM | TREND(sparkline) | QUAD | ...    │    │
│  │ - 點擊 toggle visibility                 │    │
│  │ - 未來: hover → ComparePanel overlay     │    │
│  └──────────────────────────────────────────┘    │
│                                                  │
│  Charts: Plotly (RRG) + D3 (brush) +             │
│          lightweight-charts (sparklines)          │
└──────────────────────┬──────────────────────────┘
                       │ HTTP
┌──────────────────────┴──────────────────────────┐
│                FastAPI Server                    │
│  GET  /api/baskets                               │
│  GET  /api/rrg/{basket}?trail=252&source=ibkr    │
│  POST /api/rrg/custom                            │
│  GET  /api/prices/compare                        │
│  GET  /api/dp/{symbol}        (未來: DP levels)  │
└──────────────────────┬──────────────────────────┘
                       │
┌──────────────────────┴──────────────────────────┐
│              Python Compute Engine               │
│  src/rrg/                                        │
│  ├── compute.py   JdK RS-Ratio/Momentum          │
│  ├── data.py      orchestrator + cache            │
│  ├── config.py    baskets + IBKR config           │
│  ├── providers/   IBKR + yfinance                 │
│  └── render.py    靜態 WebP/HTML (CLI 用)         │
└─────────────────────────────────────────────────┘
```

## 數據流

```
User interaction
    │
    ├── Basket dropdown / Custom input
    │       → GET /api/rrg/{basket}?trail=252
    │       → GET /api/prices/compare
    │       → 回傳: RRGResponse (252 個 RRG points + close 價格)
    │       → 回傳: PriceSeriesResponse (benchmark 折線)
    │
    ├── Tail slider (1-63)
    │       → 純前端: 改 windowPoints → brush 寬度變
    │       → RRG chart slice: points[endIdx - tail + 1 ... endIdx]
    │       → 不打 API
    │
    ├── Brush drag
    │       → 純前端: D3 brush → xScale.invert → Date
    │       → handleWindowChange → 找最近 RRG point index
    │       → 更新 windowEndIdx → 重新 slice
    │       → 不打 API
    │
    └── Source/Timeframe toggle
            → 重新打 API (source=ibkr|yf, timeframe=daily|weekly)
```

## 參數統一

| 概念 | 前端 | API param | Python |
|---|---|---|---|
| 顯示幾個 RRG 點 | `tailLength` (state) | `trail` | `trail_length` |
| Brush 右邊 index | `windowEndIdx` (state) | — | — |
| 數據源 | `source` (state) | `source` | `source` |
| 時間框架 | `timeframe` (state) | `timeframe` | `timeframe` |
| API 固定拉全量 | — | `trail=252` | — |

## 組件職責

| 組件 | 職責 | 圖表庫 |
|---|---|---|
| `BenchmarkLine` | benchmark 折線 + D3 brush window | D3 SVG |
| `RRGChart` | 四象限散點圖 + trail + hover popup | Plotly |
| `SymbolTable` | symbol 列表 + metrics + sparkline | — |
| `Sparkline` (待建) | 小型走勢圖 (table + hover 共用) | lightweight-charts |
| `ComparePanel` (待建) | 多股比較折線 overlay | lightweight-charts |
| `TailSlider` | 控制 tail length | native input |
| `ControlBar` | Source / Timeframe 切換 | native buttons |
| `SymbolInput` | 自訂 symbols 輸入 | native input |
| `BasketSelector` | basket dropdown | native select |

## 未來功能路線

### Phase B (當前 → 下一步)
- [ ] `Sparkline` 組件: lightweight-charts, 100×24px
- [ ] Table TREND 欄: 填入真實 sparkline
- [ ] RRG hover popup: 填入真實 sparkline
- [ ] ComparePanel: hover 展開多股比較
- [ ] TradingView ↗ 按鈕: `window.open` 到完整圖表

### Phase C: 算法擴展
- [ ] compute.py strategy pattern (JdK → 可插拔)
- [ ] MACD-on-RS 動量軸
- [ ] KDJ-on-RS 動量軸 (1h 適用)
- [ ] API `?method=jdk|macd|kdj` 參數

### Phase D: Dark Pool 整合
- [ ] DP 數據接入: stock_vault skill `stv:darkpool`
  - 數據來源: CheddarFlow via Haiku + chrome extension
  - 輸出: `03_docs/research/tickers/{SYMBOL}/darkpool/dp-{date}.md`
  - 格式: frontmatter 含 `top_level_1/2/3` + last_price
- [ ] API endpoint: `GET /api/dp/{symbol}` → 讀 vault 的 DP markdown
- [ ] RRG marker shapes:
  - ▲ 三角: 價格在大 DP 下方 (有壓力)
  - ◆ 菱形: 價格在 DP 附近 (關鍵位)
  - ★ 星形: 價格已突破 DP (機構確認)
- [ ] Sparkline DP overlay: `createPriceLine` 虛線

### Phase E: 多頁面 / 進階 UI
- [ ] React Router 多頁面
- [ ] Flow panel (自訂組合管理)
- [ ] Slider 動畫播放 (自動回放旋轉歷史)
- [ ] 並排快照導出 (三張不同時間點)
- [ ] Obsidian iframe 嵌入
- [ ] 公開部署 (給群友用)

## 檔案結構

```
rrg/
├── src/rrg/              # Python 核心 (10 個模組, 全部 <200 行)
│   ├── __init__.py
│   ├── compute.py         # JdK 算法 + classify_quadrants
│   ├── config.py          # baskets + IBKR config
│   ├── data.py            # orchestrator + cache + fallback
│   ├── output.py          # 結構化輸出 + manifest
│   ├── render.py          # 靜態圖片 (CLI 用)
│   ├── api.py             # Python API for skills
│   ├── cli.py             # CLI 入口
│   └── providers/
│       ├── __init__.py    # DataProvider ABC
│       ├── yfinance.py
│       └── ibkr.py
├── server/                # FastAPI
│   ├── app.py
│   ├── routes.py
│   └── schemas.py
├── frontend/              # React + TS
│   └── src/
│       ├── App.tsx
│       ├── api/client.ts
│       ├── hooks/
│       │   ├── useRRGData.ts
│       │   └── useBrush.ts
│       ├── components/
│       │   ├── RRGChart.tsx
│       │   ├── BenchmarkLine.tsx
│       │   ├── SymbolTable.tsx
│       │   ├── TailSlider.tsx
│       │   ├── ControlBar.tsx
│       │   ├── SymbolInput.tsx
│       │   ├── Sparkline.tsx      # 待建
│       │   └── ComparePanel.tsx   # 待建
│       ├── types/rrg.ts
│       └── styles/theme.ts
├── docs/
│   ├── 00-phases.md
│   ├── 01-algorithm.md
│   ├── 02-design-decisions.md
│   ├── 03-implementation.md     # 本文件
│   └── target_ui/              # StockCharts 參考截圖
├── baskets.toml
├── pyproject.toml
└── CLAUDE.md
```

## 開發規範

- 每個 Python 檔案 < 200 行 (有 hook 強制)
- Commit 前: `ruff check` + `tsc --noEmit`
- Commit 需 codex GPT-5.5 或 Opus 4.6 review
- 前後端分離: server/ 和 frontend/ 獨立
- API 回傳 JSON, 前端做所有 slicing/filtering
