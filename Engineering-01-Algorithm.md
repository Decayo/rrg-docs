# RRG Algorithm Research

## 核心概念

RRG 是一個 2D 散點圖，X 軸 = RS-Ratio（相對強度），Y 軸 = RS-Momentum（動量），中心 (100, 100)。
Securities 在四象限之間旋轉：Leading → Weakening → Lagging → Improving → Leading。

---

## 算法對比（必須全部實作後比對輸出）

### Method A: WMA 比值法（LuxAlgo / Pine Script）

來源：TradingView LuxAlgo Relative Strength Scatter Plot

```python
rs = close_sym / close_bench
wma_rs = WMA(rs, length=20)
rs_ratio = WMA(rs / wma_rs, length=20) * 100
rs_momentum = rs_ratio / WMA(rs_ratio, length=20) * 100
```

特點：簡潔、無歸一化、輸出自然圍繞 100；但不同 symbol 散佈範圍不標準化。

### Method B: Z-Score 歸一化法（OpenBB / 學術）

```python
rs = close_sym / close_bench
rs_smooth = EMA(EMA(rs, 10), 10)
rolling_mean = rs_smooth.rolling(window).mean()
rolling_std = rs_smooth.rolling(window).std()
rs_ratio = (rs_smooth - rolling_mean) / rolling_std * scale + 100
rs_momentum = (rs_ratio / rs_ratio.shift(m) - 1) * scale + 100
```

特點：Z-score 讓不同 symbol 可比、學術上嚴謹、需要 calibrate scale 和 window。

### Method C: SMA Crossover 法（RRG-Lite）

```python
rs = close_sym / close_bench
sma_short = SMA(rs, 10)
sma_long = SMA(rs, 30)
rs_ratio = (sma_short / sma_long - 1) * scale + 100
rs_momentum = (rs_ratio / rs_ratio.shift(1) - 1) * scale + 100
```

特點：作者實測後選擇此法，認為 EMA 方法 "too volatile"。最穩定。

### Method D: JdK 原版近似（StockCharts 逆向）

```python
rs = close_sym / close_bench
jdk_rs = EMA(EMA(rs, 10), 10)
rs_ratio = 100 + k * (jdk_rs - mean(jdk_rs, m)) / std(jdk_rs, m)
roc = rs_ratio / rs_ratio.shift(1)
rs_momentum = 100 + k * (roc - mean(roc, m)) / std(roc, m)
# k ≈ 5-10, m ≈ 52 (週) / 252 (日)
```

特點：最接近官方 RRG，但 k/m 需要 calibrate。

---

## 實作順序

1. 全部實作四種方法（compute.py 裡作為策略）
2. 用相同數據（SPY sectors, 6M daily）跑四種，產出四張圖
3. 去 StockCharts 截圖對比，選最接近的作為 default
4. 其他保留作為 `--method` 參數

---

## 數據需求

**Phase 1（標準 RRG）**: Daily Close only, 至少 1 年歷史, yfinance
**Phase 2（多因子）**: RSI / Volume / IV / DP / GEX 作為 overlay（點大小/顏色/邊框）

多因子歸一化：`z * scale + 100`（統一到 RRG scale）

---

## 時間維度

| 模式 | 一個點 = | 適用 |
|------|---------|------|
| 日線 | 一天 | 短線 swing |
| 週線 | 一週 | 中線趨勢 |
| 雙窗口 | W1: 4w→2w, W2: 2w→now | 加速/減速對比 |

尾跡 = 最近 N 點軌跡，預設 5。

---

## 展現層選項

| 層 | 格式 | 誰看 | Phase |
|----|------|------|-------|
| WebP/PNG | 靜態圖片 (CLI) | Claude 讀圖 | 1 (legacy) |
| HTML | Plotly 互動 (CLI) | 人用瀏覽器 | 1 (legacy) |
| Web UI | Canvas + D3 brush + React | trader 主用 | 1 ✅ 已是主入口 |
| Obsidian | DataviewJS + Chart.js/D3 | vault dashboard | 2 (未實作) |

架構：`compute → JSON → frontend Canvas render`（CLI render.py 為 admin tool，見 [R4 in refactor-backlog](31-refactor-backlog.md)）

---

## 參考

- [StockCharts RRG](https://chartschool.stockcharts.com/table-of-contents/chart-analysis/chart-types/relative-rotation-graphs-rrg-charts)
- [RRG-Lite](https://github.com/BennyThadikaran/RRG-Lite)
- [OpenBB RRG](https://openbb.co/blog/building-a-relative-rotation-graph-with-openbb/)
- [relativerotationgraphs.com](https://relativerotationgraphs.com/educational/the-building-blocks-for-rrg/)
- Pine Script 本地：`stock_vault/99_others/scripts/pinescript/RRG/template-RRG.pine`
