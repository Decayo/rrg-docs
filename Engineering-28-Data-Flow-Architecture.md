# OrbitFlow Data Flow Architecture

> 2026-05-10。工程規範文件。所有時間相關開發必須遵守。

## 核心原則

### 1. Unix Timestamp Everywhere

```
Provider (IBKR/yfinance/Polygon)
  ↓ unix timestamp (int, seconds)
Backend compute (pandas)
  ↓ unix timestamp (int, seconds)  
API response (JSON)
  ↓ unix timestamp (int, seconds)
Frontend state (React)
  ↓ unix timestamp (int, seconds)
Render layer ONLY → browser timezone display
```

**除了渲染/顯示層，全部用 Unix timestamp (seconds) 傳遞。**

- Backend：`pd.DatetimeIndex` 內部用 datetime64，但 API 輸出統一轉 Unix seconds
- API JSON：`date` 欄位統一用 `number`（Unix seconds），不用 string
- Frontend：保持 number 直到最後渲染時才 `new Date(ts * 1000).toLocaleString()`
- lightweight-charts：直接吃 Unix seconds（UTCTimestamp format）

### 2. Provider 抽象（可換源）

```
DataProvider (ABC)
  ├── IBKRProvider      ← 主要（real-time，需 TWS/Gateway）
  ├── YFinanceProvider  ← fallback（免費，有延遲限制）
  ├── PolygonProvider   ← 未來
  └── ThetaDataProvider ← 未來
```

每個 provider 只負責：
- 接收 `(symbols, period, interval)` → 回傳 `pd.DataFrame`
- 內部處理 API 特有的 date 格式、barSize mapping、whatToShow 等
- **上層代碼不知道也不關心是哪個 provider**

### 3. Brush（時間窗）永遠固定

```
Brush = daily benchmark 2 年
不隨 interval 變化
不隨 source 變化
永遠是 SPY (或用戶的 benchmark) 的 daily close
```

## 現有問題（待修）

### Bug 1: IBKR index alignment

IBKR 逐 symbol fetch → 每個 symbol dates 不完全一致 → DataFrame outer join → NaN → RRG 聚集

**Fix:** IBKR provider 內部 `ffill()` 對齊 index

### Bug 2: 混合 date format

現在 API response 用 string（`"2026-05-01"` 或 `"2026-05-01T14:30"`），應該統一 Unix seconds

### Bug 3: services.py config 混雜

`_NORM_WINDOW`, `_INTERVAL_PERIOD`, `_TRAIL_BARS`, `_EMA_PERIOD` 四個 dict hardcode 在 services 裡

## 目標架構

### 數據流

```
┌─────────────────────────────────────────────────┐
│ Data Providers                                  │
│  IBKRProvider.fetch_close(syms, period, interval)│
│  → pd.DataFrame (DatetimeIndex, float columns)  │
│  YFinanceProvider (fallback, same interface)     │
└──────────────────┬──────────────────────────────┘
                   │ DatetimeIndex
┌──────────────────▼──────────────────────────────┐
│ Data Layer (src/rrg/data.py)                    │
│  fetch_prices() → cache → provider → fallback  │
│  Output: pd.DataFrame (aligned, clean, ffilled) │
└──────────────────┬──────────────────────────────┘
                   │ DatetimeIndex
┌──────────────────▼──────────────────────────────┐
│ Compute Layer (src/rrg/compute.py)              │
│  compute_rrg() → RRGResult                     │
│  (rs_ratio, rs_momentum as DataFrames)          │
└──────────────────┬──────────────────────────────┘
                   │ DatetimeIndex
┌──────────────────▼──────────────────────────────┐
│ Service Layer (server/services.py)              │
│  build_rrg_response() → RRGResponse            │
│  *** DatetimeIndex → Unix seconds (int) ***     │
│  This is the ONLY place dates convert           │
└──────────────────┬──────────────────────────────┘
                   │ Unix seconds (int)
┌──────────────────▼──────────────────────────────┐
│ API Layer (JSON response)                       │
│  { "date": 1746086400, "rs_ratio": 105.3, ... } │
└──────────────────┬──────────────────────────────┘
                   │ Unix seconds (int)
┌──────────────────▼──────────────────────────────┐
│ Frontend State (React)                          │
│  RRGPoint.date = number (Unix seconds)          │
│  OHLCVPoint.date = number (Unix seconds)        │
│  PricePoint.date = number (Unix seconds)        │
└──────────────────┬──────────────────────────────┘
                   │ Unix seconds (int)
┌──────────────────▼──────────────────────────────┐
│ Render Layer ONLY                               │
│  lightweight-charts: 直接吃 UTCTimestamp        │
│  D3 brush: new Date(ts * 1000)                  │
│  Text labels: toLocaleString() 用瀏覽器時區     │
└─────────────────────────────────────────────────┘
```

### Interval 對 RRG 的影響

| 影響 | 怎麼變 |
|------|--------|
| Data fetch period | 由 interval 決定（15m→30D, 1d→2Y, 1wk→2Y）|
| Bar size | 由 interval 決定（15m bars, daily bars, weekly bars）|
| EMA period | 由 interval 決定（15m→80, 1d→10, 1wk→5）|
| Norm window | 由 interval 決定（15m→100, 1d→252, 1wk→52）|
| Trail length | 由 interval 決定（15m→400, 1d→63, 1wk→13）|
| **Brush** | **不變（永遠 daily 2Y）** |
| **Sparkline** | **不變（永遠 daily）** |
| **%D/%W/%R** | **不變（永遠 daily）** |

### Source 切換

```python
# 用戶 settings 決定 source
source = settings.source  # "ibkr" | "yf"

# Provider 自動 fallback
try:
    provider = get_provider(source)  # ibkr
except:
    provider = get_provider("yf")    # fallback
    # UI 顯示 "YF ⚠" 警告
```

## 遷移步驟

### Phase 1: 修 IBKR 1D（現在）
1. ibkr.py 加 ffill() 對齊 index
2. 驗證 1D RRG 分散正常

### Phase 2: 統一 Unix timestamp（之後）
1. services.py: `d.strftime(...)` → `int(d.timestamp())`
2. schemas.py: `date: str` → `date: int`
3. frontend types: `date: string` → `date: number`
4. toChartTime: 不再需要（直接是 number）
5. BenchmarkLine: `new Date(ts * 1000)` 替代 `new Date(dateStr)`

### Phase 3: SRP 重構（M2）
1. 提取 cache 層
2. 提取 config
3. services 拆分
