# MVP1 UX Spec — Basket 管理 + Top Bar 重設計

> 此文件是 E2 EPIC 的 UX 規格，需用戶確認後才進入技術實作。

---

## Top Bar 新佈局

```
┌──────────────────────────────────────────────────────────────────────┐
│ ← ◉◉◉◉◉  │  Core Macro Cross-Asset (SPY) ▼  │  Tail ━━●━━ 21     │
│ back orbit │        basket title / custom     │  3d 1w 2w 1m 3m    │
│    bar     │        + dropdown                │                    │
├────────────┼──────────────────────────────────┼────────────────────┤
│            │                                  │  [EMA] □ △ | 9seq  │
│            │                                  │  strategy selector │
│            │                                  │  + pattern markers │
├────────────┼──────────────────────────────────┼────────────────────┤
│            │                                  │  YF·D  [DU] ▼     │
│            │                                  │  badge  user menu  │
└──────────────────────────────────────────────────────────────────────┘
```

### 左側區域
| 元素 | 功能 | MVP1 |
|------|------|------|
| **← Back** | 回到上一層 RRG（drill-down 返回） | ✅ |
| **Orbit Bar** | 圓圈 icon 列，快速切換自訂 basket | ✅ |

### 中間區域
| 元素 | 功能 | MVP1 |
|------|------|------|
| **Basket Title** | 動態顯示當前 basket 名稱 + benchmark | ✅ |
| **Basket Dropdown** | 點擊 → 展開完整 Basket Panel | ✅ |

### 右側區域
| 元素 | 功能 | MVP1 |
|------|------|------|
| **Strategy Selector** | 算法切換（EMA/KDJ/等） | placeholder |
| **Pattern Markers** | 9seq, C&H, W底, M頭等型態標記 | placeholder |
| **Source·Timeframe Badge** | YF·D / IBKR·W | ✅ 已做 |
| **UserMenu** | DU avatar + dropdown | ✅ 已做 |

---

## Basket Panel（完整檢視）

觸發方式：Basket Dropdown 旁按鈕 **或** 右鍵 RRG 圖

```
┌───────────────────────────────────────────────────────┐
│                    Basket Panel                        │
│                                                        │
│  ┌──────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │  最近使用  │  │  推薦 groups  │  │  自定義          │  │
│  │          │  │              │  │                 │  │
│  │ • Macro  │  │ • 今日熱門    │  │  [+ 新建]       │  │
│  │ • Sectors│  │ • AI 推薦    │  │  [🔍 搜尋]      │  │
│  │ • Tech   │  │              │  │  [🤖 AI 建立]   │  │
│  │ • Custom1│  │              │  │                 │  │
│  │          │  │              │  │  My Baskets:    │  │
│  │ (stack)  │  │              │  │  • 我的持倉      │  │
│  │          │  │              │  │  • 幣圈觀察      │  │
│  └──────────┘  └──────────────┘  └─────────────────┘  │
│                                                        │
│  三個獨立 component，可復用，付費可 disable              │
└───────────────────────────────────────────────────────┘
```

### 三欄設計
| 欄 | 內容 | 付費限制 |
|----|------|---------|
| **最近使用** | 最近切換的 basket，latest on top（stack） | 無 |
| **推薦 groups** | 當天推薦（基於市場狀態） | 免費可看，付費可用 |
| **自定義** | 搜尋框 + AI 建立 + 用戶的 saved baskets | 免費 1 個，付費無限 |

---

## Create Basket 流程

```
[+ 新建] 或 [🤖 AI 建立]
    │
    ├─ 手動建立 ─────────────────────────────────┐
    │   1. 輸入 basket 名稱                       │
    │   2. 選擇 benchmark (SPY/QQQ/...)           │
    │   3. 搜尋 + 加入標的                        │
    │      └─ Symbol 搜尋（候選下拉）             │
    │         MVP1: 美股 + 台股代碼 + 主流幣種     │
    │   4. [儲存]                                  │
    └─────────────────────────────────────────────┘
    │
    ├─ AI 建立（MVP1 最小版）─────────────────────┐
    │   1. 上傳持倉截圖                           │
    │   2. AI 解析 ticker list                    │
    │   3. 確認 + 編輯                            │
    │   4. 命名 + 儲存                            │
    └─────────────────────────────────────────────┘
```

### Symbol 搜尋候選
- MVP1 方案：後端 API `/api/symbols/search?q=XL`
- 數據源：靜態 JSON（美股 ~8000 tickers + 台股代碼 + 幣種）
- 前端：debounce 300ms + 下拉候選列表

---

## Drill-down 流程

```
Layer 0: SPY vs [XLK, XLF, XLE, XLI, XLB, XLU, XLV, XLY, XLP, XLRE, XLC]
              │
              │ 點擊 XLK → DetailPanel → [標的 RRG ↗] 按鈕
              ▼
Layer 1: XLK vs [AAPL, MSFT, NVDA, AVGO, ...前十大]
              │
              │ ← Back → 回到 Layer 0
              ▼
         (暫定兩層，僅預設大 ETF 有 drill-down)
```

### ETF 持股數據
- 來源：`pandas.read_html` 爬 stockanalysis.com 或 hardcode
- 存儲：SQLite weekly snapshot（ticker, weight_pct, etf, snapshot_date）
- 前十大：按市值/權重排序取 top 10

### Back Navigation
- 方案待定：URL state (`?basket=sectors&layer=0`) vs 內部 state stack
- 需要：basket history stack，push on drill-down，pop on back

---

## Drill-down 入口（DetailPanel 內）

**位置：** DetailPanel header，timeframe buttons 右側

```
TLT  Lagging  $85.30  %D %W %R  |  15M 1H [1D] 1W  [⤢] [TV] [─] [×]
                                                      ↑
                                                 drill-down
                                                 icon: Maximize2 或 Network
                                                 tooltip: "展開標的 RRG"
                                                 (僅大 ETF 顯示此按鈕)
```

**點擊行為：**
1. 關閉 DetailPanel
2. push 當前 basket + brush state 到 history stack
3. 載入新 RRG（benchmark = 該 symbol，symbols = 前十大持股）
4. **繼承 brush window 日期範圍**（非 index，是實際日期）
5. Top bar: ← Back 亮起，Title 更新
6. 用戶拖動 brush 後 → 該 layer 有自己的 brush state
7. ← Back 時恢復上一層的 brush state

**History Stack 結構：**
```ts
interface RRGLayer {
  basket: { benchmark: string; symbols: string[]; name: string };
  brushWindow: { start: string; end: string };  // ISO date
  tailLength: number;
}
// stack: RRGLayer[]，push on drill-down，pop on back
```

---

## Strategy Selector + Pattern Markers（placeholder）

RRG 圖標題右側：

```
[EMA ▼] [□ 形態] [△ 形態]  |  [9seq] [C&H] [W] [M] ...
────────────────────────────  ─────────────────────────
  算法選擇器（MVP2+）          型態標記（MVP2+）
  可切換 EMA/KDJ/自訂          tail 上的視覺註解
```

MVP1：顯示 placeholder UI（灰色 disabled），點擊提示 "Coming in Pro"。

---

## 專有名詞定義

| 術語 | 英文 | 說明 |
|------|------|------|
| Orbit Bar | Orbit Bar | 左上角圓圈 icon 快速切換列 |
| Basket Panel | Basket Panel | 三欄完整檢視面板 |
| Strategy Selector | Strategy Selector | 算法選擇（EMA/KDJ） |
| Pattern Markers | Pattern Markers | tail 上的型態標記（9seq/C&H/W/M） |
| Back Nav | Back Nav | drill-down 返回上一層 |
| Drill-down | Drill-down | ETF → 成份股 RRG |

---

## MVP1 實作範圍

### ✅ 要做的
1. Basket CRUD（手動建立 + 儲存 + 刪除）
2. Basket Panel（三欄：recent / recommended placeholder / custom）
3. Orbit Bar（快速切換）
4. Symbol 搜尋候選（後端 API + 靜態 JSON）
5. Drill-down 兩層（大 ETF → 前十大持股）
6. Back Nav（history stack）
7. DB model（baskets table: user_id, name, benchmark, symbols, created_at）
8. AI 建立 basket（截圖 → ticker list，最小版）

### 🔲 Placeholder（顯示 UI 但 disabled）
9. Strategy Selector
10. Pattern Markers
11. 推薦 groups 欄

### ❌ 不做（MVP2+）
12. 條件篩選（quadrant filter, momentum threshold）
13. 自建 ETF / synthetic ticker
14. 券商連動
15. 跨市場（台股/幣種資料源）
16. AI 聊天窗 + 對話歷史
