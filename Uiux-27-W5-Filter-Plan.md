# W5: Filter + Tag Integration Plan

> 2026-05-08。W5 專屬實施計劃。
> 大計劃見 `docs/24-mvp1-implementation-plan.md`。

## Scope 精簡

研究後發現：
- **CandlestickChart 已有 RSI/KDJ/MACD sub-panes** → Detail Panel 重做推遲 M2
- **Pattern detection + Badge 系統** → 推遲 M2（依賴後端 pattern detector）
- **MVP1 聚焦 Filter Panel** → 核心差異化功能，後端全部就緒

## 任務

| # | 任務 | 複雜度 |
|---|------|--------|
| W5.1 | `api/tag-client.ts` — 前端 tag API client | 小 |
| W5.2 | `hooks/useFilterTags.ts` — tag filter state hook | 中 |
| W5.3 | `components/FilterPanel.tsx` + `FilterPill.tsx` | 中 |
| W5.4 | App.tsx 整合（tag → hiddenSymbols + table 置灰） | 中 |
| W5.5 | basket_tag_states 持久化 | 小 |
| W5.6 | 上限 1000 | 小 |

推遲到 M2：Detail Panel 重做、Badge/pill 系統、Color tag（E2.15-17）

## 新增檔案（4 個）

| 檔案 | 行數 | 用途 |
|------|------|------|
| `api/tag-client.ts` | ~50 | fetchAllTags, fetchBasketTagStates, updateBasketTagStates |
| `hooks/useFilterTags.ts` | ~80 | enabled tags 管理 + 計算 filteredOut symbols |
| `components/FilterPanel.tsx` | ~120 | Floating card: sector pills + color pills + reset |
| `components/FilterPill.tsx` | ~40 | 單個 toggleable pill |

## 修改檔案

| 檔案 | 改動 |
|------|------|
| `App.tsx` | useFilterTags + FilterPanel + filteredOut 傳遞 |
| `SymbolTable.tsx` | filteredOut prop → 置灰 + 排底 |

## 實作順序

```
1. tag-client.ts
2. useFilterTags.ts
3. FilterPill.tsx + FilterPanel.tsx
4. App.tsx 整合
5. basket_tag_states 持久化
6. 上限 1000
```

## 驗證

- tsc + vitest 通過
- FilterPanel 顯示 sector pills，點 pill → RRG 隱藏不匹配
- 切 basket → filter 恢復
- 建立 500+ symbol basket 不報錯
