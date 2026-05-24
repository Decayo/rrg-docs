# 日期/時區架構設計

> 此文件定義數據管線中日期的 normalize 策略。
> 所有涉及日期的開發必須先讀此文件。

---

## 一、問題域

| 數據源 | 市場 | 原生時區 | 日期格式 | 交易時間 |
|--------|------|---------|---------|---------|
| yfinance | US | ET (UTC-4/5) | pandas Timestamp | 09:30-16:00 ET |
| yfinance | TW | CST (UTC+8) | pandas Timestamp | 09:00-13:30 CST |
| yfinance | JP | JST (UTC+9) | pandas Timestamp | 09:00-15:00 JST |
| IBKR API | 任意 | exchange local | Unix epoch | 各市場不同 |
| Polygon | US | ET | ISO 8601 | 09:30-16:00 ET |
| 幣圈 | 全球 | UTC | ISO 8601 | 24/7 |
| 未來自建 | 任意 | 需定義 | 需定義 | 需定義 |

**核心挑戰：** 同一天在不同時區是不同的日期。
NYSE 5/1 收盤 (16:00 ET) = 台灣 5/2 04:00。
如果用戶同時看 US + TW 股票，"5/1" 對兩個市場意義不同。

---

## 二、Canonical Date Format（內部統一格式）

### Daily
```
"YYYY-MM-DD"
```
- 含義：**該交易所的本地交易日**
- NYSE 的 "2026-05-01" = 美東時間 5/1 的交易日
- TWSE 的 "2026-05-01" = 台北時間 5/1 的交易日
- **不做跨時區轉換** — 保留 exchange local date

### Intraday
```
"YYYY-MM-DDTHH:mm"
```
- 含義：**該交易所的本地時間**
- NYSE 的 "2026-05-01T14:30" = 美東 14:30
- TWSE 的 "2026-05-01T10:30" = 台北 10:30
- **前端顯示時需要標註時區**（ET / CST / JST）

### 儲存/傳輸
- API response 的 `date` 欄位 = canonical format
- 不帶 timezone offset（不是 `"2026-05-01T14:30-04:00"`）
- timezone 資訊放在 **metadata**（basket/symbol 層級），不在每個 data point 裡

---

## 三、Normalize 職責鏈

```
┌─────────────────────────────────────────────────────────┐
│ Layer 1: Provider（數據源適配器）                         │
│                                                          │
│  yfinance_provider.py                                    │
│    → 拿到 pandas Timestamp（可能帶 timezone）             │
│    → normalize: ts.tz_localize(None) 去掉 tz info        │
│    →           ts.strftime("%Y-%m-%d")  (daily)          │
│    →           ts.strftime("%Y-%m-%dT%H:%M") (intraday)  │
│    → 輸出: 純字串，exchange local time                    │
│                                                          │
│  ibkr_provider.py                                        │
│    → 拿到 epoch seconds                                  │
│    → 轉成 exchange local time 字串                       │
│                                                          │
│  polygon_provider.py（未來）                              │
│    → 拿到 ISO 8601 帶 offset                             │
│    → 轉成 exchange local time 字串                       │
├─────────────────────────────────────────────────────────┤
│ Layer 2: Data Orchestrator（data.py / services.py）      │
│                                                          │
│  不做日期轉換                                             │
│  信任 Provider 層已 normalize                             │
│  只做 cache key 和 merge                                 │
├─────────────────────────────────────────────────────────┤
│ Layer 3: API Response（routes / schemas）                │
│                                                          │
│  date 欄位 = canonical format 字串                       │
│  額外欄位: exchange_tz: "America/New_York" (metadata)    │
├─────────────────────────────────────────────────────────┤
│ Layer 4: Frontend（dateUtils.ts）                         │
│                                                          │
│  不做 timezone 轉換                                       │
│  直接用後端的字串做比較/filter/snap                       │
│  toChartTime: daily→字串, intraday→epoch                 │
│  顯示時加 timezone 標籤（從 metadata 讀）                │
└─────────────────────────────────────────────────────────┘
```

**關鍵原則：** Normalize 只在 Provider 層做一次，之後所有層直接用字串。

---

## 四、跨市場日期對齊

### Daily — 不做對齊
US 和 TW 的 "5/1" 是各自的交易日，不需要對齊。
RRG 圖上 US 的 "5/1" 點和 TW 的 "5/1" 點是同一天的不同市場。

### Intraday — 需要標註
如果同時顯示 US 和 TW 的 intraday 數據：
- US "14:30" = ET, TW "10:30" = CST
- 前端需要顯示 timezone label
- **不做轉換到用戶 timezone**（交易者習慣看 exchange time）

### 幣圈 — 特殊處理
- 24/7 無收盤概念
- 日線以 UTC 00:00 分割
- 前端可選擇以用戶 timezone 或 UTC 顯示

---

## 五、區間策略

| 類型 | 區間 | 原因 |
|------|------|------|
| windowDates | **閉區間 [start, end]** | brush 選中的首尾日都包含 |
| filterByDateRange | **閉區間** | 和 windowDates 一致 |
| snapStart | **>= target** | marker 不超出左邊界 |
| snapEnd | **<= target** | marker 不超出右邊界 |
| OHLCV K 棒 | **左閉右閉** | 每根 K 棒代表該時段的完整數據 |

---

## 六、前端 toDateKey 為什麼用 local getters

```
D3 brush 產生 Date 物件
  → new Date("2026-05-01") 在 UTC+8 = May 1 08:00 local
  → toISOString() = "2026-05-01T00:00:00.000Z" → slice = "2026-05-01" ✓
  
但 D3 brush 拖動產生的 Date 是從 xScale.invert(pixel) 來的
  → 可能是 "May 1 00:00 local" (UTC+8)
  → toISOString() = "2026-04-30T16:00:00.000Z" → slice = "2026-04-30" ✗ 偏移!
  
所以用 getFullYear/getMonth/getDate (local getters) 確保不偏移。
```

---

## 七、未來 TODO

- [ ] Provider 層統一 normalize 介面
- [ ] API response 加 `exchange_tz` metadata
- [ ] 前端 intraday 顯示加 timezone label
- [ ] 幣圈 UTC/local 切換
- [ ] 跨市場 RRG 的日期對齊策略（非交易日怎麼處理）
- [ ] 5min 精度支持（dateUtils filterByDateRange intraday 模式）
