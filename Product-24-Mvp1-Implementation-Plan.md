# MVP1 Implementation Plan

> 2026-05-07 更新。對齊 doc/26（Claude Design prototype）的 Phase 順序。
> MVP1 目標：**群友能註冊使用、你能勉強盯盤、能部署上線。**
> 架構一步到位（避免重構），UI 基於 Claude Design prototype 實作。

---

## 一、開發階段與依賴

```
W1: DB + Tag Schema           ← ✅ done
 │
W2: Tag API + Seed            ← ✅ done
 │
W3: Focus Layout 重做          ← ✅ done
 │
W4: 核心導航                    ← ✅ done
 │
W5: Filter + Tag               ← ✅ done
 │
W6: Login + Auth               ← ✅ done（i18n → M2）
 │
W6b: 15min 數據 + RRG 算法     ← ✅ done（5/6, D10 KDJ 待做）
 │
W6c: 多股並列 Finviz grid      ← 🔨 NEXT
 │
W7: Deploy                     ← Cloudflare + 域名 + Tunnel
 │
W8: 收尾                       ← 群友測試 + onboarding
```

> W = Wave，不是 Week。每個 Wave 完成後 checkpoint。
> 設計參考：`docs/26-design-to-implementation.md` + `docs/design-prototypes/`

---

## 二、Wave 詳細任務

### W1: 地基 — DB + Auth ✅

| # | 任務 | 狀態 |
|---|------|------|
| W1.1 | SQLite → PostgreSQL (psycopg3 + pool) | ✅ |
| W1.2 | Tag schema 8 張表 | ✅ |
| W1.3 | basket_symbols 關聯表（Option B） | ✅ |
| W1.4 | Auth 骨架（email/password + dev bypass） | ✅ 差自助註冊 → W6 |
| W1.5 | Docker Compose + Makefile | ✅ |

---

### W2: 數據層 — Tag API + Seed ✅

| # | 任務 | 狀態 |
|---|------|------|
| W2.1 | Tag CRUD API (routes + schemas) | ✅ |
| W2.2 | System tag seed (18 sector + 16 color) | ✅ |
| W2.3 | tag_translations (zh-TW + en) | ✅ |
| W2.4 | symbol_tags 初始綁定 | ✅ |
| W2.5 | i18n 前端框架 | → W6 |
| W2.6 | 10 integration tests | ✅ |

---

### W3: Focus Layout 重做 ✅

| # | 任務 | 狀態 |
|---|------|------|
| W3.1 | FloatingPanel 原語組件 | ✅ drag + opacity + glass |
| W3.2 | App 佈局重寫 | ✅ RRG hero + floating panels |
| W3.3 | ~~Toolbar 重做~~ | ⚠️ 文字 tabs 版本，**不符 B1 orbit design** → W4b 修 |
| W3.4 | SymbolTable 改 floating | ✅ 左下 floating card |
| W3.5 | DetailPanel 改 floating | ✅ click toggle + bottom bar |

---

### W4: 核心導航 — 部分完成

| # | 任務 | 狀態 |
|---|------|------|
| W4.1 | Spotlight ⌘K（A1 三欄） | ✅ SpotlightDialog |
| W4.2 | Basket Browser (B1 popover) | ✅ BasketBrowserPopover |
| W4.3 | Basket Create Drawer (C) | ✅ BasketDrawer + ManualTab |
| W4.4 | Symbol multi-paste | ✅ useSymbolPaste hook |
| W4.5 | PRO Lock mask | → M2 |

#### W4b: Toolbar 對齊 Prototype ✅

| # | 任務 | 狀態 |
|---|------|------|
| W4b.1 | ◎ Brand-mark + OrbitFlow 文字 | ✅ CSS circle + accent dot |
| W4b.2 | Nav tabs (Discover/Watchlist/Alerts/Notes) | ✅ Discover active，其他 placeholder |
| W4b.3 | Basket dropdown (Name + BENCH + ▼) | ✅ 觸發 BasketBrowserPopover |
| W4b.4 | Search bar → Spotlight | ✅ 功能保留，刪重複 search |
| W4b.5 | Layout + Bell icons | ✅ placeholder |
| W4b.6 | Gradient avatar (兩字母) | ✅ |
| W4b.7 | 清理 CreateBasketDialog | ✅ 已刪除 |
| W4b.8 | Popover 三欄修正 | ✅ Recommended/Recent/My baskets |

#### W4c: RRG 互動效果 ✅

| # | 任務 | 狀態 |
|---|------|------|
| W4c.1 | Hover dim（漸變淡出其他 symbols） | ✅ lerp 0.3, ~80ms |
| W4c.2 | Click glow（呼吸脈動整條 trail + dot） | ✅ 3 層亮色 glow, sine wave |
| W4c.3 | symbolGlow()（同 hue 高 lightness） | ✅ HSL 82% light, 50% sat |

---

### W5: Filter + Tag ✅

| # | 任務 | 狀態 |
|---|------|------|
| W5.1 | Tag API client | ✅ tag-client.ts |
| W5.2 | useFilterTags hook | ✅ |
| W5.3 | FilterPanel + FilterPill | ✅ 收合式 TAG FILTER bar |
| W5.4 | App.tsx 整合 | ✅ mergedHidden + table filteredOut |
| W5.5 | basket_tag_states 持久化 | 🔲 API ready，前端待接 |
| W5.6 | 上限 1000 | ✅ |
| W5.7 | Table tooltips (shadcn Tooltip) | ✅ |
| W5.8 | Table focused row + scroll | ✅ |

**推遲到 M2：** Detail Panel 重做、Badge/pill 系統、Color tag dot ring/quick-tag

---

### W6: Login + Auth ✅

| # | 任務 | 狀態 |
|---|------|------|
| W6.1 | Login 重做 (v3 design) | ✅ |
| W6.2 | 自助註冊 | ✅ |
| W6.3 | Google OAuth | ✅ |
| W6.4-6.5 | i18n | → M2 |
| W6.6 | Settings | ✅ |
| W6.7 | 品牌統一 | ✅ |

---

### W6b: 15min 數據 + RRG 算法 ← 🔨 NOW

| # | 任務 | 狀態 |
|---|------|------|
| W6b.1 | D4: 時間軸 floating overlay | ✅ U40 三色 timeline tag (blue viewport / orange RRG / grey crosshair) |
| W6b.2 | D8: 股票全名顯示（hover + detail） | ✅ DockedChart header 右上 fullName + RRG tooltip |
| W6b.3 | D1: IBKR 15min provider（backend interval 參數化） | ✅ OHLCV 走 provider chain (IBKR primary, yf fallback) |
| W6b.4 | D2: Cache TTL 分 interval（15min→5min, daily→10min） | ✅ _MEM_TTL_BY_INTERVAL + response cache 5min |
| W6b.5 | D9: Baseline timeframe 架構（全局切換 + plan_tier gate） | ✅ 5y fetch + trail_bars + on-demand tile |
| W6b.6 | D10: KDJ-based RRG 算法（method 參數切換） | 🔲 |

---

### W6c: 多股並列

| # | 任務 | 狀態 |
|---|------|------|
| W6c.1 | D3: Finviz grid（2 欄滾動，K+量+價格，無 RSI/KDJ/MACD） | 🔲 |

---

### W7: 部署

| # | 任務 | 產出 | Done When |
|---|------|------|-----------|
| W5.1 | 域名購買 | orbitflow.xxx 或類似 | DNS 指向 Cloudflare |
| W5.2 | Cloudflare 設定 | DNS + Pages（前端）或 Tunnel（全走 tunnel） | HTTPS 通 |
| W5.3 | 後端部署 | Arch 自 host：Docker Compose (FastAPI + PG) + Cloudflare Tunnel | API 可從公網訪問 |
| W5.4 | PostgreSQL 備份 | pg_dump cron + 本地/S3 | 每日備份確認可恢復 |
| W5.5 | 前端部署 | Cloudflare Pages 或 Tunnel serve static | 域名打開能用 |
| W5.6 | 環境變數 | JWT_SECRET, PG_URL, GOOGLE_CLIENT_ID 等 | 不 hardcode |

---

### W6: 收尾

| # | 任務 | 產出 | Done When |
|---|------|------|-----------|
| W6.1 | E2E 測試 | 註冊 → 建 basket → filter → 盯盤 完整流程 | 手動 + 自動化通過 |
| W6.2 | 效能檢查 | 500 node basket 不卡、Filter 切換 < 100ms | Chrome DevTools 確認 |
| W6.3 | Error handling | API 錯誤有 toast、網路斷線有提示 | 不 silent fail |
| W6.4 | Onboarding | 首次登入預設 basket 自動載入，象限有簡單說明 | 新手打開不迷路 |
| W6.5 | 品牌統一 | 頁面 title、favicon、logo 統一為 OrbitFlow / OrbitMap | 對外一致 |

---

## 三、明確不做（MVP2+）

| 功能 | 排入 |
|------|------|
| AI chat + 截圖辨識 | MVP2 |
| Notes 系統 | MVP2 |
| cap / beta / signal / custom tag 前端露出 | MVP2 |
| Pseudo symbol / 合成 ETF | MVP2 |
| 定價 + 付費 gate | MVP2 |
| DP levels / 進階指標 | MVP3 |
| 13F 持倉匯入 | MVP3 |
| Mobile RWD 適配 | MVP2 |
| zh-CN 翻譯 | MVP2 |
| Tag 管理 UI（新增/刪除自訂 tag） | MVP2 |
| Mute mode（灰化保留 vs 消失） | MVP2 |
| Long-press solo mode | MVP2 |
| Detail Panel 重做（pattern tags on candles） | MVP2 |
| Badge/pill 系統（pattern glyph vocabulary） | MVP2 |
| Color tag dot ring + quick-tag modal | MVP2 |
| PRO Lock mask | MVP2 |
| 台股支援（時區校正 + 代碼標準化 + FinMind） | MVP2 |
| i18n（react-i18next + zh-TW 翻譯） | MVP2 |
| 財報日期 tag + 提前提醒 | MVP2 |
| 盤前盤後夜盤數據 | MVP2 |
| Chatbot 大改（markdown + 多輪 + streaming） | MVP2 |
| RabbitMQ + Redis message queue | MVP2 |

---

## 四、架構預留（MVP1 建好但不露出）

這些表/欄位在 W1-W2 就建好，確保未來不需 migration：

- `tags.namespace` 支援 `cap`, `beta`, `signal`, `custom`（MVP1 只用 `sector` + `color`）
- `tag_translations` 表（MVP1 只填 zh-TW + en）
- `basket.type` 支援 `portfolio`, `synthetic`（MVP1 全部 `watchlist`）
- `holdings_json` 內 `weight` 欄位（MVP1 全 null）
- `notes` 表（MVP1 不做 UI，但表已存在）

---

## 五、依賴 & 風險

| 風險 | 影響 | 緩解 |
|------|------|------|
| Google OAuth Client ID 申請延遲 | Auth 只有 email/password | 先做 email，OAuth 並行申請 |
| 台股數據源不穩 | 台灣用戶體驗差 | Yahoo Finance 2330.TW 先測，備案找 FinMind |
| PostgreSQL 遷移複雜度 | W1 耗時超預期 | 用 Alembic，舊 SQLite 資料先不遷（dev 重建） |
| 1000 node 效能 | Canvas OK 但 table/API 可能慢 | Virtualized table（react-virtual），API 分頁 |
| Cloudflare Tunnel 設定 | 從沒弄過 | 文檔清楚，worst case 先用 ngrok |

---

## 六、文件關係

| 文件 | 角色 |
|------|------|
| **本文件 (doc/24)** | MVP1 唯一的實施計劃，所有開發以此為準 |
| `docs/22` | 歷史參考（UX 概念），被本文件取代 |
| `docs/23/` | 架構參考（Tag spec + 產品定位），設計理念來源 |
| `docs/13` | 全量 backlog，MVP1 完成後更新狀態 |
