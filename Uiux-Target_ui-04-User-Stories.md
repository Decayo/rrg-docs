# MVP1 User Stories — Chrome Extension 測試用

---

## US-1: 基本瀏覽（已完成功能的驗證）

```
前置：用戶已登入（dev bypass）

1. 打開 app → 看到預設 US Sectors basket
2. 看到 top bar: Orbit icon + basket dropdown + Tail slider + YF·D badge + UserMenu
3. 看到 Benchmark Line (SPY) + RRG 四象限圖 + Symbol Table
4. 拖動 Brush → RRG + Table 同步更新 timeframe
5. 調整 Tail slider → trail 長度變化
6. Hover RRG 圓點 → 顯示 tooltip (symbol + RS 值 + sparkline)
7. Click RRG 圓點 → 打開 DetailPanel (K 線 + GMMA + RSI/KDJ/MACD)
8. DetailPanel 可拖曳、可縮放、可折疊
9. 切換 DetailPanel timeframe (15M/1H/1D/1W) → K 線重新載入
10. 點 TV 按鈕 → 新分頁開 TradingView
11. 點 × → 關閉 DetailPanel
```

**驗證點：** 全流程無 console error，數據載入正常，互動流暢。

---

## US-2: Drill-down（ETF → 成份股）

```
前置：正在看 US Sectors (SPY vs XL 系列)

1. 點擊 XLK → 打開 DetailPanel
2. 看到 DetailPanel header 有 [⤢] drill-down 按鈕
   - 非 ETF 的 symbol（如個股 AAPL）不顯示此按鈕
3. 點擊 [⤢] →
   a. DetailPanel 關閉
   b. RRG 重新渲染：XLK vs [AAPL, MSFT, NVDA, AVGO...]（前十大持股）
   c. Top bar title 更新為 "XLK vs Top Holdings"
   d. ← Back 按鈕出現
   e. Brush window 日期範圍保持不變（如 2/25~3/27）
   f. Orbit Bar 不變
4. 在 Layer 1 拖動 Brush → 日期範圍改為 1/15~2/28
5. 點 ← Back →
   a. 回到 SPY vs XL 系列
   b. Brush window 回到 2/25~3/27（Layer 0 自己的記憶）
   c. ← Back 按鈕消失（已在最頂層）
6. 點 Orbit icon（左上角品牌 logo）→ 回到首頁（和 Layer 0 相同）
```

**驗證點：** drill-down 正確載入持股、timeframe 繼承、back nav 恢復上一層 state。

---

## US-3: Basket 快速切換（Orbit Bar）

```
前置：用戶有 3 個自訂 basket

1. 看到 Orbit Bar: [←] [S] [M] [🔥] [...] [+]
   - S = Sectors（預設首字母）
   - M = Macro
   - 🔥 = 自訂 emoji
   - [+] = 快速新建 basket（打開 CreateBasketForm）
2. 點擊 [M] → RRG 切換到 Macro basket
   - 下方 * 指示器移到 M
   - Title 更新
3. 超過顯示範圍 → 出現 [...] 按鈕
   - 當前選中的 basket 如果在 [...] 裡，指示器指向 [...]
4. 點 [...] → 展開完整列表（或打開 Basket Panel）
5. 點 [+] → 打開 CreateBasketForm
```

**驗證點：** 切換流暢、指示器跟隨、overflow 處理正確。

---

## US-4: Basket Panel（完整檢視 + 新建）

```
前置：用戶已登入

1. 觸發 Basket Panel:
   - Desktop: 點 dropdown 旁的按鈕 / 右鍵 RRG
   - Mobile: 長按 RRG
2. 看到三欄面板:
   a. 最近使用 — stack（latest on top）
   b. 推薦 groups — placeholder（MVP1 顯示 "Coming soon"）
   c. 自定義 — 搜尋框 + [+ 新建] + AI 建立 + 我的 baskets 列表
3. 免費用戶看到自定義欄有 lock icon + 模糊 + "Pro 解鎖"
4. 付費用戶（或測試者）看到完整功能
5. 點擊最近使用中的 basket → 切換 RRG，Panel 關閉
6. 搜尋框輸入 "semi" → 篩選含 "semi" 的 basket 名稱或標的
```

**驗證點：** 三欄顯示、付費限制 UI、搜尋篩選、切換正確。

---

## US-5: 手動建立 Basket

```
前置：打開 Basket Panel → 自定義欄 → [+ 新建]

1. 打開 Create Basket 介面:
   a. 輸入名稱: "半導體觀察"
   b. 選擇 benchmark: SMH（下拉 + 搜尋）
   c. 搜尋標的: 輸入 "NV" → 候選下拉出現 "NVDA - NVIDIA Corp"
   d. 點擊 NVDA → 加入列表
   e. 繼續加入 AMD, TSM, AVGO, ASML
   f. 可選 emoji icon（預設首字母 "半"）
2. 點 [儲存] →
   a. Basket 存到 DB
   b. Orbit Bar 新增一個圓圈
   c. RRG 切換到新建的 basket
3. Symbol 搜尋候選:
   - 輸入延遲 300ms debounce
   - 顯示: ticker + 公司名
   - MVP1: 美股 ~8000 tickers
```

**驗證點：** 搜尋候選正常、basket 存到 DB、Orbit Bar 更新。

---

## US-6: Basket 管理（編輯/刪除）

```
入口方式 A: 長按 Orbit Bar 的 basket icon → 進入編輯介面
入口方式 B: UserMenu 旁邊的籃子 icon → GroupBasketViewPanel

GroupBasketViewPanel（總覽）:
1. 看到所有 baskets 列表（預設 + 自訂）
2. 預設 basket 不可刪除、不可編輯
3. 自訂 basket 可點擊 → SingleBasketEditPanel

SingleBasketEditPanel（單一編輯）:
1. 可修改名稱
2. 可修改 benchmark
3. 可新增/移除標的
4. 可修改 emoji icon
5. 可刪除整個 basket（二次確認）
6. [儲存] → 更新 DB + Orbit Bar
```

**驗證點：** 兩個入口都能進入、CRUD 完整、預設不可刪。

---

## US-7: Settings（已部分完成）

```
1. 點 UserMenu → Settings → SettingsSheet 右側滑出
2. Data section: 切換 Source (YF/IBKR) → RRG 自動重新載入
3. Data section: 切換 Timeframe (Daily/Weekly) → RRG 自動重新載入
4. Display section: 切換 Label mode → RRG labels 即時變化
5. Appearance section: Theme (Dark only, Light "coming soon") 
6. Appearance section: Language (en/zh-TW)
7. 關閉 Sheet → 設定已自動存到 API + localStorage
```

**驗證點：** 切換即時生效、持久化正確。

---

## US-8: 認證流程

```
1. 未登入 → 訪問 / → 自動跳轉 /login
2. Login 頁: Orbit icon + 品牌名 + "Sign in to continue"
3. 有 Google Sign-In 按鈕（需 Client ID 配置）
4. 有 Skip Login (Dev) 按鈕（開發環境）
5. 點 Skip Login → JWT 存入 localStorage → 跳轉 /
6. 刷新頁面 → token 自動驗證 → 不需重新登入
7. UserMenu → Logout → 清 token → 跳轉 /login
```

**驗證點：** AuthGuard 攔截、token 持久化、logout 清除。

---

## US-9: Symbol Table 互動

```
1. Click row → toggle 該 symbol 在 RRG 的可見性
2. Double-click row → 打開 DetailPanel
3. 點 [⤢] 展開 icon → 打開 DetailPanel
4. 點 [★] → toggle favorite（置頂 + localStorage 持久化）
5. 點 [TV] → 新分頁 TradingView
6. 點 header → 3-state 排序（none → desc → asc）
7. Benchmark row (SPY) 置頂、不可 toggle、不可 favorite
```

**驗證點：** 每個互動都正確、favorite 重新整理後保留。

---

## US-10: AI 建立 Basket（MVP1 最小版）

```
前置：AI 聊天窗已整合（使用 pal MCP 或 pplx API）

1. Basket Panel → 自定義欄 → [🤖 AI 建立]
2. 打開 AI 對話介面（簡易版）
3. 上傳持倉截圖
4. AI 解析出 ticker list: AAPL, MSFT, TSLA, NVDA...
5. 顯示解析結果 → 用戶可編輯/確認
6. 自動填入 Create Basket 表單
7. 命名 + 儲存

技術: 先用 pal MCP 頂著，rate limit 20秒/請求，5人測試
未來: pplx API + function calling
```

**驗證點：** 截圖上傳、AI 回應、ticker 解析正確、可編輯後儲存。

---

## 行為補充：Nav 系統

```
← Back    → 回上一層 RRG（Layer N → Layer N-1）
→ Forward → 回下一層 RRG（如果有 history）
Orbit icon（品牌 logo）→ 回首頁（Layer 0，預設 basket）

Layer 0: ← 和 → 都隱藏
Layer 1+: ← 顯示，→ 在有 forward history 時顯示
```

---

## 組件清單（從 user stories 衍生）

| Component | 對應 US | 狀態 |
|-----------|---------|------|
| OrbitBar | US-3 | 🔲 新建 |
| BasketPanel (三欄) | US-4 | 🔲 新建 |
| CreateBasketForm | US-5 | 🔲 新建 |
| SymbolSearch (候選下拉) | US-5 | 🔲 新建 |
| GroupBasketViewPanel | US-6 | 🔲 新建 |
| SingleBasketEditPanel | US-6 | 🔲 新建 |
| DrilldownButton (DetailPanel 內) | US-2 | 🔲 新建 |
| BackForwardNav | US-2 | 🔲 新建 |
| AIChatPanel (簡易版) | US-10 | 🔲 新建 |
| StrategySelector (placeholder) | — | 🔲 placeholder |
| PatternMarkers (placeholder) | — | 🔲 placeholder |
