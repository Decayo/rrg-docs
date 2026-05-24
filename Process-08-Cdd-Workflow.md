# CDD — Component-Driven Development 工作流

## 方法論

每個 UI 組件遵循：**隔離建構 → 驗證 → 封存 → 組裝**。

組件一旦通過驗證就不再修改內部邏輯，只透過 props 控制行為。
上層改動不影響下層已封存的組件。

## 工具鏈

| 工具 | 用途 |
|---|---|
| **Dev Playground** (`/dev` 路由) | 隔離渲染每個組件，可切換 props/mock data |
| **Claude Code Chrome Extension** | 瀏覽器視覺驗證 + 互動測試（hover/click/scroll） |
| **Vitest** | 邏輯單元測試（compute 函數、hook state 轉換） |
| **tsc --noEmit** | 型別正確性（pre-commit hook 已有） |

### 不用 Playwright / Storybook

- Playwright → 被 Claude Code Chrome Extension 取代（已有 MCP 工具，能截圖、點擊、hover、讀 DOM）
- Storybook → 被 Dev Playground 取代（輕量，不需要額外 build pipeline）

## 開發流程

```
1. 建組件
   └─ 在 components/ 建新 .tsx
   └─ 只接 props，無 side effect

2. 加入 Playground
   └─ DevPlayground.tsx 加一個 Section
   └─ 餵 mock data，涵蓋正常/邊界情況

3. 視覺驗證（Claude Code Chrome）
   └─ 開 /dev 頁面
   └─ 截圖確認渲染正確
   └─ 測試互動：hover → tooltip、click → panel、resize → responsive

4. 單元測試（Vitest）
   └─ 計算邏輯：輸入 → 輸出
   └─ Hook 狀態轉換
   └─ 不測 DOM（chrome extension 做這件事）

5. 封存
   └─ 組件文件頂部加 @sealed 註解
   └─ 意味著：props interface 穩定，內部實作已驗證
   └─ 未來只改 bug，不改行為

6. 組裝
   └─ 在 App.tsx 或父組件中使用
   └─ 只需驗證組裝邏輯（props 傳遞、事件串接）
   └─ 組件內部已封存，不需重新驗證
```

## Dev Playground 規格

路徑：`/dev`（開發環境限定，production build 排除）

```tsx
// DevPlayground.tsx
<Section title="Sparkline" description="SVG 走勢小圖">
  <Sparkline points={mockUp} width={80} height={24} />      // 漲
  <Sparkline points={mockDown} width={80} height={24} />     // 跌
  <Sparkline points={mockFlat} width={80} height={24} />     // 平
  <Sparkline points={[]} width={80} height={24} />           // 空資料
</Section>

<Section title="HoverTooltip" description="RRG 圓點懸停提示">
  <HoverTooltip info={mockHoverLeading} />    // Leading 象限
  <HoverTooltip info={mockHoverLagging} />    // Lagging 象限
</Section>

<Section title="DetailPanel (Sheet)" description="點擊展開的 stacked panel">
  <SheetTrigger> → <DetailPanel symbol="AAPL" /> </SheetTrigger>
</Section>
```

每個 Section：
- 標題 + 簡述
- 多個 variant（正常/邊界/錯誤狀態）
- 深色背景（和 app 一致）

## Chrome Extension 驗證 Checklist

每個組件驗證時，Claude Code 執行：

```
□ 截圖 — 視覺正確（尺寸、顏色、文字）
□ Hover — tooltip/highlight 出現在正確位置
□ Click — panel 展開、狀態切換
□ Resize — 不溢出、不跑版
□ Edge case — 空資料不 crash、超長文字不破版
□ 暗色主題 — 確認用了 CSS variable 不是 hardcoded
```

## 組件封存狀態追蹤

| 組件 | 狀態 | 驗證日期 | 備註 |
|---|---|---|---|
| BasketSelector | ✅ 封存 | — | 已有，穩定 |
| ControlBar | ✅ 封存 | — | 已有，穩定 |
| TailSlider | 🔄 待遷移 | — | 遷移到 shadcn Slider |
| BenchmarkLine | ✅ 封存 | — | D3 brush，穩定 |
| RRGChart | ✅ 更新 | 2026-04-29 | label toggle + trail 粗細漸變 + hover sparkline |
| SymbolTable | ✅ 更新 | 2026-04-29 | TREND 欄 placeholder → 真實 Sparkline |
| Sparkline | ✅ 封存 | 2026-04-29 | @sealed. 9 tests. SVG polyline, 漲綠跌紅 |
| LabelToggle | ✅ 封存 | 2026-04-29 | @sealed. 輕量 popover, 5 模式 |
| HoverTooltip | ✅ 整合 | 2026-04-29 | 嵌入 RRGChart，非獨立組件 |
| DetailPanel | ⬜ placeholder | 2026-04-29 | 佈局骨架完成，待填入 lightweight-charts |
| DevPlayground | ✅ 完成 | 2026-04-29 | /dev 路由，Sparkline section |
