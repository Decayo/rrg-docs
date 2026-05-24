# E11: RRG Trail Signal System

> 2026-05-20。需求規格。RRG tail 上的視覺信號層 + Price Note + Signal Dock。
> 算法參考：`docs/ref_pine/CC-yata.pine`（CC Yata Sequential Count）

---

## 一、概述

在 RRG trail 和 K chart 上同步渲染交易信號。所有信號 CRUD 化（function call ready）。
統一的 pill 渲染模型 + hover 互動 + floating Signal Dock 控制顯隱。

### 三類信號

| # | 類型 | 來源 | RRG trail | K chart |
|---|------|------|-----------|---------|
| 1 | Sequential Count (9/13) | Backend compute | ▲ pill + 數字 | bar marker |
| 2 | Price Note | 用戶/AI 建立 | pill + 價位 | priceLine 橫線 |
| 3 | GMMA 聚攏+觸碰 | Backend compute | 回踩 pill | EMA 群交叉 marker |

---

## 二、Trail Pill 渲染模型（統一）

### 2.1 Pill 定義

Pill = 一條固定長度的短線段，**永遠垂直於 trail 在該點的切線方向**。

```
                   ▲9          ← 三角形 + 數字（pill 上方）
                    │
        ●━━━━━━●━━━╋━━━●━━━━●  ← trail（曲線）
                    │
                  $135         ← 價位文字（pill 下方）

        切線方向 →→→
        pill 方向 ⊥ 切線
```

因為 trail 是曲線，每個 pill 的角度隨曲率變化。

### 2.2 Pill 規格

| 屬性 | 值 |
|------|-----|
| 長度 | 固定 pixel（如 24px），不隨 zoom 縮放 |
| 線寬 | 比 trail 粗（trail 1px → pill 2-3px） |
| 文字 | 最多 4 字元。超過縮寫：2413 → `2.4k`，$135.20 → `$135` |
| 顏色 | **跟隨 symbol 的 color tag**（和 RRG dot ring 同色系） |

### 2.3 切線計算

```typescript
// trail points: [{x: rs_ratio, y: rs_momentum}, ...]
// 在 point[i] 處的切線：
const tangent = {
  x: points[i+1].x - points[i-1].x,
  y: points[i+1].y - points[i-1].y,
};
// 法線（垂直於切線）：
const normal = { x: -tangent.y, y: tangent.x };
// 正規化到 pill 長度：
const len = Math.sqrt(normal.x ** 2 + normal.y ** 2);
const unit = { x: normal.x / len, y: normal.y / len };
// pill 端點（screen coords）：
const pillStart = { x: screenX + unit.x * PILL_HALF, y: screenY + unit.y * PILL_HALF };
const pillEnd   = { x: screenX - unit.x * PILL_HALF, y: screenY - unit.y * PILL_HALF };
```

### 2.4 堆疊

同一 trail 點有多個信號時：

```
正常態：
    ●━━╋━━●    只顯示最高優先的 pill + badge (⁺2)
        │

hover 態：
    ●━━╋━━●    沿 normal 方向展開所有 pill
       ╋
       ╋

click → 常駐 panel on RRG（點其他地方才關）
```

**優先序：** Price Note > Signal (Seq/GMMA)
**排列：** 按 trail 時間順序（和 trail icon list 一致）

### 2.5 Hover / Click 互動（共用模組）

**和現有 RRG dot hover 是同一個模組體系。**

現在 RRG dot 的 hover 已經是完整模組：tooltip 顯示 symbol、sparkline、window 數據。
Pill 的 hover/click 是同一個模組的 instance，差異只在內容（信號類數據 vs symbol 數據）。

```
共用模組: TrailTooltip
  ├─ RRG dot hover（現有）→ symbol + sparkline + quadrant + %D/%W
  └─ Pill hover（新）→ signal type + date + price + detail
      同樣的 tooltip 容器、定位邏輯、動畫
```

**互動：**
- hover → tooltip panel（和 dot hover 同風格，含 sparkline + signal 詳情）
- click → 常駐 panel（點其他地方才關）
- hover 區域 = pill bounding box + padding
- 堆疊態 hover → 展開全部 pill，各自獨立 hover 區域

---

## 三、Signal 1: Sequential Count（CC Yata）

### 3.1 算法（從 Pine Script 移植）

參考：`docs/ref_pine/CC-yata.pine`

#### Setup（計數到 9）

```python
# Sell setup: close > close[4] 連續計數
sell_set = 0
if close > close[4]:
    sell_set = (prev_sell_set + 1) if prev_sell_set < 9 else 1
else:
    sell_set = 0

# Buy setup: close < close[4] 連續計數
buy_set = 0
if close < close[4]:
    buy_set = (prev_buy_set + 1) if prev_buy_set < 9 else 1
else:
    buy_set = 0
```

Setup 9 完成 = 信號觸發。

#### Countdown（計數到 13）

Setup 9 完成後開始。不需連續，累計滿足條件即可。

```python
# Standard mode:
# Buy countdown: close < low[2]
# Sell countdown: close > high[2]

# Aggressive mode:
# Buy countdown: low < low[2]
# Sell countdown: high > high[2]
```

#### Support / Resistance

```python
# Setup 9 完成時：
# Buy setup 9 → resistance = highest(9)
# Sell setup 9 → support = lowest(9)
# 突破後失效（close > resistance 或 close < support）
```

#### Stealth 9

```python
# Setup 8 完成後 1 bar 內方向反轉
stealth_buy = bars_since(buy_set == 8) <= 1 and sell_set == 1
stealth_sell = bars_since(sell_set == 8) <= 1 and buy_set == 1
```

#### 非合格 13 (+)

```python
# Standard countdown 12 → 條件滿足但 low > countdown_8_close
# 標記為 "+" 而非 "13"
```

### 3.2 顯示規則

**RRG trail：**
- 只標 9 和 13（setup 完成和 countdown 完成）
- ▲ = buy signal（三角朝上），▼ = sell signal（三角朝下）
- 數字在三角形內部
- Stealth 9 標為 `s9`
- 非合格 13 標為 `+`

**K chart：**
- lightweight-charts markers API
- bar 下方 = buy signals，上方 = sell signals
- Support/Resistance = priceLine 水平線

### 3.3 顏色

Pine 原版漸變（opacity 從高到低表示序號增加）：

| 序號 | Buy 色 | Sell 色 |
|------|--------|---------|
| 1-5 | `#1C61B1` → `#1EA9AC` (藍→青) | `#69208E` → `#CA3AB0` (紫→粉) |
| 6-8 | `#34BEA5` → `#67D89A` (青→綠) | `#F23D92` → `#F71746` (粉→紅) |
| 9 | `#90EE90` (亮綠) | `#E21B22` (亮紅) |
| 13 | `#2196F3` (藍，buy) / `#FF9800` (橘，sell) | — |

---

## 四、Signal 2: Note（flat + tag, Obsidian-like）

> **2026-05-20 Grill 決策：** Note 不是 enum-based「price note」，而是 flat entity，
> 由 tag 決定它是什麼。tag namespace `signal:*` 包含 support/resistance/dp_level/
> large_oi/custom 五種預設，皆已在 `seed_tags.py` 種入。

### 4.1 Data Model

```sql
-- E11 schema（已實作於 server/db.py）
CREATE TABLE notes (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  ticker TEXT,             -- 留作 pre-E11 backward compat
  symbol VARCHAR(20),      -- nullable: NULL = 全局筆記
  price NUMERIC(12,4),     -- nullable: NULL = 純文字筆記（無水平線）
  content TEXT NOT NULL DEFAULT '',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- 共用 tags 表，透過 note_tags 關聯
CREATE TABLE note_tags (
  note_id INTEGER NOT NULL REFERENCES notes(id) ON DELETE CASCADE,
  tag_id  INTEGER NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  PRIMARY KEY (note_id, tag_id)
);
```

決定 note「是什麼」靠 tag：
- `signal:support` / `signal:resistance` / `signal:dp_level` / `signal:large_oi` / `signal:custom`
- 用戶建立的自訂 tag（`custom:*`）也可掛上

### 4.2 RRG trail 渲染

Note pill 顯示該價位的縮寫（如 `$135`），渲染在 trail 末端附近，垂直於切線。
（M1 尚未實作；K chart 水平線已可用。）

### 4.3 K chart 渲染

`series.createPriceLine()` 橫線 + label，顏色由第一個 `signal:*` tag 決定：
- `signal:support` → 綠虛 `#22c55e`
- `signal:resistance` → 紅虛 `#ef4444`
- `signal:dp_level` → 紫虛 `#a855f7`
- `signal:large_oi` → 橘虛 `#f97316`
- `signal:custom` / default → 灰虛 `#9ca3af`

實作：`frontend/src/hooks/useChartNoteLines.ts`。

### 4.4 CRUD API

```
POST   /api/notes            { symbol?, price?, content, tag_ids[] }
GET    /api/notes?symbol=X   list (or all if symbol omitted)
GET    /api/notes/{id}
PUT    /api/notes/{id}
DELETE /api/notes/{id}
```

Auth 隔離：用戶只能存取自己的 note（updateNote/deleteNote 對非擁有者回 404）。

### 4.5 Function Call

```typescript
createNote({ symbol?, price?, content, tag_ids[] })
updateNote(id, { symbol?, price?, content?, tag_ids? })
deleteNote(id)
```

**M1 AI 驗證**：`POST /api/ai/function/createNote` mock endpoint 接 Grok function
call 結構，僅 `logger.info("[FC:createNote] …")` 寫 log，不做 DB 寫入。
真正建立走 `/api/notes`（等 E12 chatbot dispatcher 串好）。

---

## 五、Signal 3: GMMA 聚攏+觸碰（Phase 1）

### 5.1 EMA 群定義

```
短期群: 3, 5, 8, 10, 12, 15
長期群: 30, 35, 40, 45, 50, 60
（超長期群 120-250 暫不參與信號計算）
```

### 5.2 信號條件（Phase 1: 聚攏+觸碰）

```python
short_spread = max(short_emas) - min(short_emas)
long_spread  = max(long_emas) - min(long_emas)
gap          = min(short_emas) - max(long_emas)  # 正=分離，負=穿越

# 聚攏: 短期群收窄
is_converging = short_spread < threshold_converge  # threshold 待調

# 觸碰: 短期群最低接近長期群最高
is_touching = abs(gap) < threshold_touch

# Phase 1 信號 = 聚攏 AND 觸碰
gmma_pullback = is_converging and is_touching
```

### 5.3 未來 Phase

| Phase | 條件 | 優先 |
|-------|------|------|
| Phase 1 | 聚攏+觸碰 | 最高（目前） |
| Phase 2 | 僅聚攏 | 中 |
| Phase 3 | 僅觸碰 | 低 |

### 5.4 渲染

- RRG trail: 回踩 icon pill（如 `↩` 或 `⟲`）
- K chart: EMA 群交叉處 marker（bar 下方 signal tag）

---

## 六、K chart Signal Tag Bar（下方）

```
┌──────────────────────────────────────────┐
│  [2/18 '26]  [OrbitMap ▸]  [◂ 5/18 '26] │ ← 上方：timeframe tag（現有）
│                                          │
│  ░░░░ K chart candles + GMMA ░░░░        │
│                                          │
│  [$135]  [▲9]  [↩]                       │ ← 下方：signal tag bar（新）
└──────────────────────────────────────────┘
```

- 風格與現有 `CandlestickTimelineTags` 一致
- 上方 = timeframe（現有藍/橘），下方 = signal（新）
- Tag 碰撞時水平堆疊，同 timeframe tag 的 TAG_W 邏輯

---

## 七、Floating Signal Dock

**獨立於 Basket Dock 的另一個 floating 框。** 位置在 Basket Dock 下方，但不是同一個組件。

### 三層折疊

```
Layer 1: 最小化（一個 icon）
┌────┐
│ 📡 │  ← 點擊最上方 ▾ 按鈕展開到 Layer 2
└────┘

Layer 2: icon 列（無 description）
┌────┐
│ ▴  │  ← 縮回 Layer 1
│ ▲9 │  ← hover → 顯示 "Sequential Count" description
│ $  │  ← hover → 顯示 "Price Notes"
│ ↩  │  ← hover → 顯示 "GMMA Pullback"
│ 👁 │  ← Hide All
└────┘

Layer 2 + hover description:
┌──────────────────────┐
│ ▴                    │
│ ▲9  Sequential Count │  ← hover 時右側展開 description
│ $   Price Notes      │
│ ↩   GMMA Pullback    │
│ 👁  Hide All          │
└──────────────────────┘
```

### 互動

- **▴/▾ 按鈕**（最上方）：切換 Layer 1 ↔ Layer 2
- **hover icon**：顯示 description 文字（Layer 2 → hover 擴寬）
- **click icon**：toggle 該信號類型的顯示/隱藏（任何 layer 都能 click）
- **active 狀態**：icon 亮色 = 顯示中，暗色 = 隱藏中
- **Hide All (👁)**：master toggle，一鍵全開/全關

---

## 八、Function Call 總覽

| Function | 說明 | CRUD |
|----------|------|------|
| `createPriceNote` | 建立 price note | C |
| `updatePriceNote` | 修改 price note | U |
| `deletePriceNote` | 刪除 price note | D |
| `getPriceNotes` | 取得某 symbol 全部 price notes | R |
| `getSignals` | 取得某 symbol 所有 computed + user signals | R |
| `toggleSignalType` | 切換某類信號的顯隱 | U |

Seq Count 和 GMMA 是 computed signals（後端計算），不需要 C/U/D。
Price Note 是 user-created，完整 CRUD。

---

## 九、依賴與排序

| 項目 | 依賴 | 建議順序 |
|------|------|---------|
| Seq Count 後端 compute | OHLCV data | 1 — 純算法，無 UI 依賴 |
| GMMA 聚攏+觸碰 compute | EMA data（已有） | 1 — 同上 |
| Price Note API | DB migration | 2 — CRUD + schema |
| Trail pill 渲染 | RRGCanvas | 3 — 前端核心 |
| K chart signal tag bar | CandlestickChart | 3 — 和 trail 並行 |
| Signal Dock | Basket Dock（U52） | 4 — UI 控制面板 |
| Function call 接入 | E12 chatbot dispatcher | 5 — AI 層 |

---

## 十、待討論

- [ ] GMMA threshold 數值需要用真實數據調參
- [ ] Sequential Count 是否需要 alert（推播/Toast）？
- [ ] Price Note 是否支援 basket-level（同一 basket 所有 symbols 套用同一條線）？
- [ ] Pill 文字方向：沿 normal 排列可能在某些角度很難讀，是否需要自動旋轉到水平？
