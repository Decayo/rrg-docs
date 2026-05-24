# Refactor Backlog

> Canonical 重構議題清單，沿著 review session 持續累積。
> Session 6 起草 [30-tech-roadmap.md](30-tech-roadmap.md) 時拉這檔的條目排優先序。
>
> **每個議題格式**：問題 → Why（為什麼是議題） → Scope（影響哪些檔） → 解法方向 → Tier + Timing
>
> **Tier**：P0 立刻 / P1 本 milestone / P2 下個 milestone / P3 等到痛了再做
> **Timing**：M1 (MVP1) / M2 (MVP2) / M3 (MVP3) / never (不做)

---

## R1 — server/ 結構扁平化

**問題**
所有後端 .py 平鋪在 `server/` 根目錄，30+ 個檔案沒有 group：

```
server/
├── auth_routes.py / auth_deps.py / auth_schemas.py / jwt_utils.py / models.py
├── basket_routes.py / basket_models.py / basket_schemas.py
├── tag_routes.py / tag_models.py / tag_relations.py / tag_schemas.py / seed_tags.py
├── note_routes.py / note_models.py / note_relations.py / note_schemas.py
├── symbol_routes.py / symbol_data_routes.py
├── routes.py / etf_routes.py / settings_routes.py / ai_routes.py
├── services.py / services_cache.py
├── db.py / app.py
```

**Why**
- 新進開發者第一眼看不出哪些檔屬於同一個 domain
- `services.py` / `services_cache.py` / `seed_tags.py` 命名相似但職責不同 → 命名混淆
- `auth_*` 系列、`tag_*` 系列、`basket_*` 系列、`note_*` 系列各 3-5 個檔，視覺上失序

**Scope**
影響 ~30 個 .py 檔案，所有 import 路徑要改。但邏輯不變。

**解法方向**
按 domain 分組（每個 domain 一個子資料夾）：

```
server/
├── auth/
│   ├── routes.py / deps.py / schemas.py / jwt.py / models.py
├── basket/
│   ├── routes.py / models.py / schemas.py
├── tag/
│   ├── routes.py / models.py / relations.py / schemas.py / seed.py
├── note/
│   ├── routes.py / models.py / relations.py / schemas.py
├── symbol/
│   ├── routes.py / data_routes.py
├── rrg/                    # ← 現在叫 routes.py
│   ├── routes.py / services.py / services_cache.py
├── etf/
├── ai/
├── settings/
├── core/                   # ← 共用底層
│   ├── db.py / app.py
```

**Tier**：P1
**Timing**：M2（MVP1 上線後馬上做，趁 codebase 還沒長更大）

**前置依賴**：無，但建議和 [R3 前後端分離評估] 一起考慮（如果未來分離，server/ 自成 package 結構先想好）

---

## R2 — Swagger / OpenAPI 對齊

**問題**
FastAPI 內建 OpenAPI 自動產 Swagger UI（`/docs`）和 ReDoc（`/redoc`），但目前狀態未確認：
- 是否啟用（`FastAPI(docs_url=...)` 沒被關掉？）
- 每個 endpoint 是否有寫 `response_model=`、docstring、`tags=` 完整描述
- frontend 是否有 auto-gen TS types from OpenAPI spec
- frontend `api/client.ts` 是手刻還是 generated

**Why**
- **單一 source of truth** 對 contract 失準預防最有效（你的 RRG 已經有 docs 失準問題）
- 未來 [R3 前後端分離] 後，OpenAPI 對齊是**唯一**讓兩個 repo 不漂移的機制
- AI 開發（GitNexus / Claude Code）對齊 spec 比對齊 code 更穩
- 用戶（你）可以直接在 `/docs` 試 API，不用寫 curl

**Scope**
- backend：FastAPI app config + 每個 endpoint 補 `response_model` / docstring
- frontend：考慮 `openapi-typescript` 或 `kubb` 從 spec 自動產 TS types + client
- CI/CD：build 時 dump spec，比對 frontend types 是否同步

**解法方向**
1. **驗證階段**（5 分鐘）：跑起 backend → 開 http://localhost:8000/docs，看 Swagger UI 出來沒
2. **對齊階段**（中期）：每個 endpoint 補 `response_model=Pydantic.BaseModel`
3. **自動化階段**（長期）：frontend npm script 跑 `openapi-typescript http://localhost:8000/openapi.json > src/api/types.gen.ts`

**Tier**：P1（驗證階段 P0）
**Timing**：M1（驗證 5 分鐘現在就能做）→ M2（對齊 + 自動化）

**前置依賴**：無

---

## R3 — 前後端 repo 分離評估

**問題**
目前 monorepo（`rrg/` 含 `server/` + `frontend/`），未來規模變大可能要分成：
- `rrg-backend`（FastAPI + algorithms）
- `rrg-frontend`（React + Vite）

**Why 用戶提出**
- codebase 越來越大（500+ 行 backlog、51 組件、30+ 後端檔）
- 直覺感覺「該分」

**權衡**

| 維度 | Monorepo（現況） | 分離 repo |
|------|----------------|-----------|
| API contract 同步 | ✅ 同 PR 改前後端 | ❌ 兩個 PR + OpenAPI publish |
| 部署 | ✅ docker-compose 一個檔 | ❌ 兩個 pipeline |
| 本地開發 | ✅ clone 一次 | ❌ clone 兩次 + 同步啟動 |
| 單獨 ship | ❌ 前端 hotfix 要碰 backend repo | ✅ 各自獨立 |
| 多 client（mobile/desktop）| ❌ frontend 跟 backend 綁太緊 | ✅ backend 純 service |
| 多人團隊 | ❌ 衝突多 | ✅ 各管各 |
| GitNexus index | ✅ 一份 graph | ❌ 兩個 index，需 group mode 跨 |
| **side project 早期** | ✅ **最快** | ❌ overhead 大 |

**業界做法**
- **保留 monorepo**：Vercel、Stripe、Linear、Anthropic claude-code（**多數 SaaS**）
- **分離**：開源框架（Django backend + 各自 frontend）
- **混合**：`pnpm workspace` / Turborepo（**邏輯分離但 repo 不分**）

**我的判斷**

**現階段（pre-MVP1 / 單人開發）強烈不建議分離**。理由：
1. API contract 還在快速變化（E11 Notes、E12 AI、E2 Tag 等大功能進行中）
2. side project 只有你一個人，分離成本 > 收益
3. 「同 PR 改前後端」是目前最大 productivity 優勢
4. Docker Compose 部署一個檔，超省事

**何時該分離（觸發條件，達到任一個就考慮）**
- ✋ 第二位開發者加入
- ✋ 後端 API contract 穩定 3 個月以上沒破壞性改動
- ✋ 要做 mobile / desktop / 第三方整合（多 client）
- ✋ 前後端部署節奏不同（前端 hotfix 不該 trigger backend）

**現在該做的折衷（不分 repo 但模擬分離的效果）**
1. 完成 [R1 server/ 結構扁平化] → 邏輯上 backend 已自成一塊
2. 完成 [R2 OpenAPI 對齊] → 未來分離時 contract 已對齊好，搬只是 git mv
3. 用 worktree workflow（Appendix Session A）做功能隔離 → 模擬「平行開發」的好處
4. `frontend/` 內已經是獨立 npm project，邏輯上已半分離

**Tier**：P3（等到觸發條件之一達到再評估）
**Timing**：never（直到觸發條件）

**前置依賴**：R1 + R2 完成後再評估

---

## R4 — `src/rrg/render.py` + `cli.py` legacy 評估

**問題**：早期 backend Plotly 渲染遺留。Web 改 Canvas 後，這兩檔只被彼此用（GitNexus impact 確認 render.py 只剩 2 個 caller：cli.py + api.py）。**Web 完全沒用**。

**Why**：
- 兩套渲染邏輯並存（Web Canvas + CLI Plotly）= 維護負擔 + 認知負擔
- 可能根本沒人跑 CLI（你還記得上次跑是什麼時候嗎？）

**Scope**：`src/rrg/cli.py` (120) + `render.py` (188) + 可能連帶 `api.py` (123) + `output.py` (60)。`compute.py` / `data.py` 留著（Web 還用）。

**解法方向**：
1. 確認還用嗎？（grep CI、Makefile、文件）
2. 用 → 標 deprecated + 文件化「admin 工具」
3. 不用 → 刪掉（GitNexus impact LOW，安全）

**Tier**：P1 評估 / P2 動工
**Timing**：M2

---

## R5 — Cache 多 worker safety + Redis 遷移

**問題**：3 層 cache 都是 process-local in-memory：
- `server/services_cache.py` (response cache, max 100, TTL 5min)
- `server/services.py` 內 `_ohlcv_cache` (LRU max 200)
- `src/rrg/data.py` 內 `_mem_cache` (**無上限** ← OOM 風險)
- 加上 `src/rrg/cache.py` disk parquet（多 worker 寫 race condition）

**Why**：
- multi-worker uvicorn → 每個 worker 各自一份 cache，記憶體浪費 + cache hit ratio 低
- `_mem_cache` 無上限 → 長跑 OOM
- file parquet 多 worker 寫衝突
- **這是 production multi-user 的真正 blocker**

**Scope**：Redis 引入 + 三層 cache 改寫

**解法方向**：
1. 短期：給 `_mem_cache` 加 LRU 上限（quick fix）
2. 中期：引入 Redis，把 response cache + OHLCV cache 都搬上去
3. 對齊 backlog **E7.6 Redis cache [M2]**（已在 product backlog 列了，但細節未定）

**Tier**：P0（production blocker）
**Timing**：M1 短期 fix + M2 Redis 完整遷移

---

## R6 — IBKR 架構約束文件化

**問題**：「backend 必須跟 IBKR 同機」是個**架構約束**（不是議題），但沒寫進文件。新人會誤以為可以把 backend deploy 到 cloud。

**Why**：IBKR TWS / Gateway 是桌面 app，跑 localhost。不能公開到 cloud（帳號端點裸奔）。架構決定 backend 必須 self-host。

**Scope**：建 `docs/engineering/architecture-constraints.md`（或加進 [21-database-decision.md](21-database-decision.md)）寫清楚：
- IBKR 為什麼必須 localhost
- Cloudflare Tunnel 公開的是 HTTP API，不是 IBKR
- 部署架構圖（user → Cloudflare → 你 Arch backend → 本機 IBKR）
- 未來如果要去掉 IBKR 依賴的選項（Alpaca / Polygon 雲端 API）

**Tier**：P1（誤導風險）
**Timing**：M1（5 分鐘寫完）

---

## R7 — `server/services.py` 198 行職責拆分

**問題**：198 行混 3 種職責：
1. RRG response 組裝（`build_rrg_response`）
2. Cache 邏輯（`_ohlcv_cache`、`cached_compute`）
3. Schema serialization（從 RRGResult 轉 JSON 格式）

**Why**：
- 改一處動全身
- 但 GitNexus impact **LOW risk**（只 2 個 file IMPORTS：routes.py + app.py）→ **拆容易，blast radius 小**

**Scope**：拆成 3 個檔：
- `services/orchestration.py` — `build_rrg_response()` 純組裝
- `services/cache.py` — `_ohlcv_cache` + cache helper
- `services/serialization.py` — RRGResult → JSON 轉換

或不拆檔，但**分 class**：`RRGComposer` + `OhlcvCache` + `Serializer`。

**Tier**：P1
**Timing**：M2

**前置依賴**：R1（server/ 結構）一起做

---

## R8 — Module-level globals 集中（Pydantic Settings）

**問題**：18+ 個檔案有 `^_[A-Z]` module-level global，沒中央管理。

| 類型 | 例子 | 出現位置 |
|------|------|---------|
| 環境變數 | `_API_KEY`, `_GOOGLE_CLIENT_ID`, `_DEV_MODE`, `_SECRET`, `_EXPIRY_SECONDS` | app.py, auth_routes.py, jwt_utils.py |
| 常數 (TTL/限制) | `_TTL_SECONDS`, `_MAX_ENTRIES`, `_OHLCV_TTL`, `_NORM_WINDOW`, `_INTERVAL_PERIOD`, `_TRAIL_BARS`, `_EMA_PERIOD` | services_cache.py, services.py |
| 路徑 | `_DATA_PATH`, `_CACHE_DIR`, `_PROJECT_ROOT` | etf_routes.py, symbol_routes.py, data.py, config.py |
| Lazy cache | `_HOLDINGS`, `_SYMBOLS` | etf_routes.py, symbol_routes.py |
| DDL 字串 | `_SCHEMA`, `_MIGRATIONS` | db.py（這個 OK，但應該抽到 schema/ 子資料夾） |

**Why**：
- 每個檔自己讀 env → 設定散落、無 validation、無預設值文件化
- 改個常數要 grep 整個 repo
- multi-worker 測試 / mock 設定困難

**解法方向**：
1. 建 `server/settings.py` 用 **Pydantic Settings**（FastAPI 標配）：
   ```python
   class Settings(BaseSettings):
       api_key: str | None = None
       google_client_id: str = ""
       jwt_secret: str = "rrg-dev-only"
       jwt_expiry_seconds: int = 7 * 24 * 3600
       ohlcv_ttl_seconds: int = 300
       # ...
   ```
2. 各 module `from server.settings import settings` 統一存取
3. DDL 字串拆到 `server/schema/{tables}.sql` 檔案

**Tier**：P2
**Timing**：M2

**前置依賴**：和 R1 一起做更合理

---

## R9 — `seed_tags.py` 改用 migration / fixture loader

**問題**：`server/seed_tags.py` 是「執行檔」(`python -m server.seed_tags`)，不是 migration。但內含必要 seed 資料：
- 16 color tags + i18n labels
- 18 sector tags（推測）
- signal tags
- 57 個 symbol-tag bindings

**Why**：
- 新環境部署 → 必須**手動**跑這個檔，DB 初始化不會自動 seed
- COLORS / COLOR_LABELS / SIGNALS 常數塞在執行檔內，不是純資料
- 沒有 idempotent guarantee（重跑會不會出事？）

**解法方向（三選一）**：
1. **抽純資料 + idempotent loader**：
   - `server/data/seed/colors.json`、`sectors.json`、`signals.json`（純資料）
   - `server/db.py` 的 `init_db()` 自動跑 seed loader（idempotent INSERT ON CONFLICT DO NOTHING）
2. **用 alembic migration**：建 alembic 後，seed 寫成 data migration
3. **保留現狀但文件化**：CLAUDE.md 寫「新環境必跑 `python -m server.seed_tags`」

**Tier**：P1（部署痛點）
**Timing**：M1（選 #1 最快）

**前置依賴**：和 R8（globals 集中）一起做

---

## R10 — 半完成 endpoint / module audit

**問題**：你提到 `etf_routes` 「卡在」。實際讀完是 lazy-load global cache 設計（work but 不優雅）。但這啟發一個更大議題：**所有「work 但半完成」的點需要 audit 列出**。

**Why**：你不知道有多少「我以為做完了但其實只是 prototype」。

**Scope**：
- `etf_routes.py` 用 `_HOLDINGS` global + lazy load + `noqa: PLW0603`（ruff 不爽 global mutation 但被壓掉了）
- `note_routes.py` UI 還沒做（backend ready）
- `ai_routes.py` 只有 chat + createNote mock，function call dispatcher 未做
- `settings_routes.py` GitNexus route_map 沒列出（要驗證有沒有真實 endpoint）
- 任何標 `TODO` / `FIXME` / `XXX` 的 code

**解法方向**：
1. grep 整個 repo `TODO|FIXME|XXX|noqa: PLW`
2. 每個落到 backlog（R10.1, R10.2...）或對應已有 E# epic
3. Session 7c 整理

**Tier**：P2
**Timing**：M2

---

## R11 — `authHeaders` 重複實作（DRY 違反，快勝）

**問題**：`authHeaders` function **在多個 API client 各自實作**：
- `frontend/src/api/tag-client.ts`（7 個 caller）
- `frontend/src/api/note-client.ts`（4 個 caller）
- `frontend/src/api/basket-client.ts`（看到 `headers` function 7 caller，疑似同問題）
- `frontend/src/api/client.ts`（已有 `authHeaders`）

**Why**：每個 client 抄一份 = 改 auth 邏輯（例如加 refresh token）要改 N 處。

**解法方向**：所有 client 統一 import `client.ts` 的 `authHeaders`。5-10 分鐘。

**Tier**：P0（快勝）
**Timing**：M1（但 review 階段不修，Session 7 之後動）

---

## R12 — `guard-200-lines.sh` 從 strict → warning

**問題**：[.claude/hooks/guard-200-lines.sh](../../.claude/hooks/guard-200-lines.sh) 對所有 .py/.ts/.tsx 200 行硬上限 deny。Review 中發現：
- shadcn primitives（dropdown-menu.tsx 268 行）天然超過
- compute.py 171 行接近上限，加新算法（如 D10 KDJ-RRG）會撞牆
- SymbolTable.tsx 197 行接近上限

**Why**：
- 200 行做 strict 是業界相當嚴格的選擇（ESLint `max-lines` default 300）
- 一旦撞牆會被迫拆檔，可能拆得更不合理
- shadcn / 算法集中檔本來就該允許超過

**解法方向**：
1. Hook 從 **deny → warning**（提示但不擋）
2. 加例外清單：
   - `components/ui/*.tsx`（shadcn primitives）
   - `src/rrg/compute.py`（算法集中）
   - `server/db.py` 的 DDL 字串（用 R8 拆 schema 後即可放鬆）
3. 真正越界 → review 階段討論而不是 hook 擋
4. **同時琢磨注入 prompt**：hook 訊息應該說明「為什麼超過上限是 smell」+ 「該怎麼判斷例外」，而不是純粹擋下

**Tier**：P0（影響開發體驗）
**Timing**：M1（立刻可改）

---

## R13 — 前端 hook / component 過細評估

**問題**：你直覺前端拆得太細。GitNexus 數據看：
- **28 個 hooks**，其中 6 個 `useChart*` 系列（useChartProps / useChartCrossMarkers / useChartGMMAMarkers / useChartNoteLines / useChartOverlays / useChartTDMarkers）—— 可能可以合併
- **51 個 components**，其中 BasketDrawer 3 個 Tab 各一檔（Manual / AI / Placeholder）—— 可能可以合併

**Why**：
- 過度拆分 = navigation 成本高（找東西要跳 5 個檔）
- 但太大也不好（無 SRP）
- 需要實際看每個拆的合理性

**Scope**：
- `useChart*` 6 個 → 評估合併成 2-3 個（按 marker type）
- `BasketDrawer*Tab` 3 個 → 評估合併成 1 個（用 prop discriminator）
- 其他疑似過細：`SpotlightCard.tsx` 內含 `SpotlightCol`、`SignalTab.tsx` 等

**解法方向**：
1. **不要急著合**——先量化：每個小檔 < 30 行？有獨立復用嗎？
2. 用 GitNexus impact 看每個小檔的 fan-in（fan-in 1 = 沒人復用 → 可合）
3. **保留 sealed 組件**（13 個 @sealed 不動）

**Tier**：P2（需先評估）
**Timing**：M2

**前置依賴**：R14（folder 分層）一起做

---

## R14 — components / hooks 按 feature 分子資料夾

**問題**：51 components 全部平鋪在 `components/`（19 ui primitives 已 group 到 ui/，但 32 業務組件無 group）。28 hooks 全部平鋪。**file explode**。

**Why**：
- 找東西靠檔名搜尋而非 folder 瀏覽
- 同 feature 的檔散落 BasketDock / BasketDrawer / BasketBrowserPopover / BasketDrawerAITab / BasketDrawerManualTab / BasketDrawerPlaceholderTab 等
- 一眼看不出「basket 系統一共多少組件」

**解法方向**：按 feature colocation（業界標準）：

```
components/
├── ui/                    # 既有 19 個 shadcn primitive
├── basket/                # BasketDock, BasketDrawer*, BasketBrowserPopover,
│                         #   CreateBasketForm, BasketSelector
├── rrg/                   # RRGCanvas, RRGTooltip, RRGIntervalSelector, BenchmarkLine
├── chart/                 # CandlestickChart, DockedChart, ChartModal,
│                         #   PatternMarkers, StrategySelector
├── filter/                # FilterPanel, FilterPill, ColorTagPopover
├── symbol/                # SymbolTable, SymbolInput
├── detail/                # DetailPanel, DetailInfo, SignalTab
├── layout/                # TopBar, AppToolbar, UserMenu, BackForwardNav
├── spotlight/             # SpotlightDialog, SpotlightCard
├── overlay/               # NoteForm, Toast, LoadingSkeleton
└── core/                  # AuthGuard, ErrorBoundary, TvButton, Sparkline,
                          #   TailSlider
```

```
hooks/
├── auth/                  # useAuth, useSettings
├── basket/                # useBaskets, useBasketActions
├── chart/                 # useChart* 6 個, useBrush, useRRGWindow, useAutoLoadMore
├── filter/                # useFilterTags, useColorTags
├── data/                  # useRRGData, useSymbolData, useSpotlightData,
│                         #   useSymbolNames, useNotes
├── ui/                    # useOverlayState, useDetailPanels, usePanelDrag,
│                         #   usePanelResize
└── core/                  # usePersistedState, useToast, useSymbolPaste,
                          #   useRRGHistory
```

**Why P1**：影響每天找東西的效率 + 未來新組件擺哪也明確

**注意**：
- 重組會破壞所有 import path（51 + 28 = 79 個檔的 import）
- 必須一次完成 + 用 codemod / VS Code refactor tool 輔助
- **可以開 worktree** 做（Appendix Session A 啟用後）

**Tier**：P1
**Timing**：M2（搬一次省一輩子，越早搬越省）

**前置依賴**：和 R13（評估過細）一起做，先評估 → 該合的合 → 再分資料夾

---

## R15 — 引入 TanStack Query（取代手刻 fetch hooks）

**問題**：28 個 hooks 中至少 6 個是手刻 fetch + loading/error state，每個都重複 boilerplate。無 cache、無 stale-while-revalidate、無 background refetch、無 deduping。

**Why（Grok outside view 強烈推薦）**：
- Trading app 靠 query caching + background refetch + stale-while-revalidate **活著**
- 切 basket 再切回來會重 fetch（無 cache）
- 並發請求不會 dedupe
- 重複的 `[loading, error, data] = useState(...)` boilerplate 至少 6 處

**Scope**：影響 `frontend/src/hooks/` 中所有 fetch hooks：
- useRRGData（最複雜，3 個 fetch 混合）
- useSymbolData
- useSpotlightData
- useSymbolNames
- useBaskets（部分）
- useNotes
- useColorTags（refresh 邏輯有 shared module cache hack）

**解法方向**：
1. `npm i @tanstack/react-query`
2. `main.tsx` 包 `<QueryClientProvider>`
3. 每個 fetch hook 改寫成 `useQuery(['key'], fetcher)` / `useMutation`
4. 自動拿到 cache / loading / error / refetch / dedupe
5. **同時順手解掉 R-13 過度拆分**（useChart\* 用 React Query 後可能合併變更乾淨）

**Tier**：**P0 stop ship**（Grok 升級）
**Timing**：M1（推 MVP1 前完成），useRRGData 為起點

**前置依賴**：無

---

## R16 — Context Provider stack 簡化（從 nesting 改 composition）

**問題**：main.tsx 5 層 nested Provider（AuthProvider > SettingsProvider > BasketsProvider > ToastProvider）→ 加新 state 會繼續往內塞。讀起來像 callback hell。

**Why（Grok 升 stop ship）**：
- Provider 順序怪：Toast 在 Settings 之外但 Toast 該最外層
- 5 層 nesting 已是上限，再加會更糟
- prop drilling 替代品本來就不好，這還是「多層 nesting」型的

**解法方向（三選一）**：

**方案 A — Composition Pattern**（推薦，最 React native）：
```tsx
function AppProviders({ children }) {
  return (
    <ErrorBoundary>
      <AuthProvider>
        <SettingsProvider>
          <BasketsProvider>
            <ToastProvider>{children}</ToastProvider>
          </BasketsProvider>
        </SettingsProvider>
      </AuthProvider>
    </ErrorBoundary>
  );
}
// main.tsx 變單層: <AppProviders><BrowserRouter>...</BrowserRouter></AppProviders>
```

**方案 B — Combiner helper**：
```tsx
const providers = [ErrorBoundary, AuthProvider, SettingsProvider, ...];
const Combined = combineProviders(providers);
```

**方案 C — Zustand 取代部分 Context**（Grok 提示）：
- Settings 和 Baskets 是純「全域 state」，用 Zustand 比 Context 更輕（無 re-render 風暴）
- Auth 留 Context（因為涉及 lifecycle）

**Tier**：**P0 stop ship**（Grok 升級，理由：在加新 state 前必須清理）
**Timing**：M1

**前置依賴**：和 R15 一起做（同時引入 TanStack Query + 簡化 Context）

---

## R17 — Table render 性能：React.memo + memoized row + useCallback

**問題（React Scan runtime 證據，2026-05-23）**：

```
FPS 67 → 10（嚴重 drop）

Ranked re-render count:
  TooltipTrigger × 40       ← 最大元兇
  ThTip × 20 (Memoizable)
  Tooltip × 20 (Memoizable)
  TooltipRoot × 20 (Memoizable)
  TooltipContent × 20 (Memoizable)
  TooltipPortal × 20 (Memoizable)
  TvButton × 8 (Memoizable)
  TvLogo × 8 (Memoizable)
```

**Why（GitNexus 確認結構）**：

```
SymbolTable.tsx (renderRow)
  ├─ 每行 ThTip × N（每個 column header tooltip）
  │   └─ ThTip → Tooltip + TooltipTrigger + TooltipContent
  └─ 每行 TvButton × 1
      └─ TvButton → TvLogo
```

任何 SymbolTable parent state 變動（hover row / sort / filter / RRG window 變）→ 整個 table re-render → renderRow 每次重新跑 → 所有 ThTip / TvButton 重建。

**根因**：
1. **ThTip / TvButton 沒 `React.memo`** → props 沒變也 re-render
2. **renderRow 是 inline function** → 不能 memoize
3. **inline event handler**（`onClick={() => ...}`）每次 parent render 是新 reference → 即使加 memo 也會失效

**Scope**：3 個檔，每個改動 < 5 行
- [frontend/src/components/ThTip.tsx](../../frontend/src/components/ThTip.tsx)（11 行）
- [frontend/src/components/TvButton.tsx](../../frontend/src/components/TvButton.tsx)（17 行）
- [frontend/src/components/SymbolTable.tsx](../../frontend/src/components/SymbolTable.tsx)（197 行）

**解法方向（3 步驟）**：

```typescript
// Step 1: ThTip 加 React.memo
export const ThTip = React.memo(function ThTip(props) { ... });

// Step 2: TvButton 加 React.memo
export const TvButton = React.memo(function TvButton(props) { ... });

// Step 3: renderRow 拆成 memoized SymbolRow component
const SymbolRow = React.memo(function SymbolRow({ symbol, onSelect, ... }) {
  return <tr>...</tr>;
});

// 同時父 component 內所有 row 用的 callback 都包 useCallback：
const handleSelect = useCallback((sym) => { ... }, [/* deps */]);
const handleToggle = useCallback((sym) => { ... }, [/* deps */]);
```

**預期效果**：FPS 從 10 → 回到 60+（trading app 流暢度=信任度）

**Tier**：**P0 stop ship**（影響體感）
**Timing**：M1（30 分鐘可修）

**前置依賴**：無

---

## R17 子項目候選（待擴展）

這個 R17 是 React Scan 跑一個 hover/interact 就抓到的。冰山一角，其他互動可能還有 FPS drop。

**該跑的 smoke 互動清單**（用 React Scan 開著）：
- [ ] hover SymbolTable rows（已知）
- [ ] sort SymbolTable column
- [ ] 點 sector filter pill（→ RRG canvas re-render）
- [ ] 切 timeframe（15m / 1h / 1d / 1wk）
- [ ] 拖 brush 改 time window
- [ ] 切 basket（OrbitBar pill 點擊）
- [ ] hover RRG canvas dot
- [ ] 點 RRG dot → DetailPanel 開啟
- [ ] 切 K chart symbol
- [ ] 開 Spotlight (⌘K) + 滑動
- [ ] 開 BasketDrawer

每個有 FPS drop 的互動 → 列為 R17.1, R17.2, ...

**根本解法（M2+）**：升級 R12（移除 200-line hook strict）+ 引入 R15（TanStack Query）+ 評估 **React Compiler**（React 19 內建，自動 memoize，可能讓所有手動 React.memo / useCallback 變多餘）。

---

## R18 — React Compiler 評估 + 啟用

**問題**：RRG 在 React 19.2.5 但**沒啟用 React Compiler**。手動 React.memo + useCallback + useMemo 是 React 17/18 時代的東西，React 19 + Compiler 時代**自動 memoize**，手動 wrapping 變多餘甚至有害（cache pressure）。

**Why**：
- R17 修法直接依賴：**沒 Compiler 修法是手動 memo；裝了 Compiler 修法是改 anti-pattern**
- frontend grep 看到 `useCallback × 99 + useMemo × 41` 的 boilerplate 大概可清掉一半以上
- 開發體驗：寫 code 不用想「該不該 memo」

**Scope**：
1. `npm i -D babel-plugin-react-compiler eslint-plugin-react-compiler`
2. [vite.config.ts](../../frontend/vite.config.ts) 加 babel plugin
3. ESLint config 加 `react-compiler` rule
4. 跑一輪驗證（Compiler 對 anti-pattern 會 bail out 不 compile，要看 console 警告）
5. 確認 lightweight-charts / Plotly / D3 等第三方相容

**風險**：
- React Compiler 目前 **RC**（2026 可能 GA）
- 對 impure code 會 bail out（需修才會 compile）
- 部分第三方 lib 不相容（例：MobX 自己有 reactive 系統，會打架）

**驗證方式**：
- ESLint plugin 會在編輯時暗示哪些地方 Compiler 不喜歡
- `console.log` 看 "[React Compiler] bailout: ..." 訊息
- React DevTools 看 component 旁邊有 `Memo ✨` 標記表示成功編譯

**Tier**：**P0 stop ship**（直接影響 R17 修法路線選擇）
**Timing**：**M1**（R17 動工前必須決定走哪條路）

**前置依賴**：無（但和 R17 必須一起決策）

**關聯**：完成後可逐步移除：
- 99 次手動 useCallback 中的大部分
- 41 次手動 useMemo 中的大部分
- 未來新 component 不再需要 React.memo wrapping

---

## R19 — Chrome GPU 加速環境文件化（Niri + Wayland + NVIDIA）

**問題（2026-05-23 React Scan 測試發現）**：
- 開發機環境：Niri compositor (Wayland-only) + NVIDIA RTX 3090
- 此組合 Chromium 系預設可能 fallback 軟體渲染
- 直接影響 FPS measurement 有效性（軟體渲染下 FPS 數字會誤導 React perf 判斷）

**Why**：
- 不文件化 → 未來新開發者 / 你自己換機器，可能誤判 React 性能問題
- 也誤導「mobile 體感」判斷（軟體渲染 ≠ mobile，mobile 有 hardware decode）

**Scope**：建 `docs/engineering/dev-env-setup.md` 寫：
- Niri / Wayland / NVIDIA 環境下 Chrome 必加 flags
- chrome://gpu 該看哪幾欄
- 真實 mobile 模擬方式（Chrome DevTools Performance CPU throttle 4x/6x）
- 對性能測試的影響（軟體渲染 vs hardware-accelerated 數字差異）

**Tier**：P2（文件型）
**Timing**：M1（5 分鐘寫完，避免未來踩坑）

**前置依賴**：無

---

## R20 — Frontend BASE URL default 8000 vs tmux dev 8666 不一致

**問題**：[frontend/src/api/auth-client.ts:6](../../frontend/src/api/auth-client.ts#L6) 預設 `BASE = "http://localhost:8000"`，但 dev tmux backend 跑 `:8666`。

**Why**：每個 api/*-client.ts 都有 `import.meta.env.VITE_API_URL ?? "http://localhost:8000"`。dev 環境若沒設 `VITE_API_URL`，frontend 會打錯 port。

**修法**：建 `frontend/.env.development` 明確設 `VITE_API_URL=http://localhost:8666`，OR default 改 `:8666`。

**Tier**：P1（馬上踩到）
**Timing**：M1（5 分修）

**參照**：sequence 01-login-flow.md 邊界議題

---

## R21 — Google token 自己造輪子（用 urllib 而非 google-auth library）

**問題**：[server/auth_routes.py:97-125](../../server/auth_routes.py#L97-L125) 用 `urllib` 打 Google tokeninfo endpoint。沒 cert pinning / 沒 retry / 沒 cache（每次 login 都 round trip Google）。

**修法**：用 `google-auth` 標準庫，內建 cache + 安全驗證 + retry。

**Tier**：P2（沒踩過事但該補）
**Timing**：M2

---

## R22 — Dev token fallback 不安全（production 前必確認）

**問題**：[server/auth_routes.py:99](../../server/auth_routes.py#L99) `if _DEV_MODE and credential == "dev-token"` — 任何人在 dev mode 發 "dev-token" 字串就能登入任何身份。

**修法**：production 部署 **must** `RRG_DEV_AUTH` unset（或 = 0）。CLAUDE.md / R6 (IBKR doc) 該明確寫部署 checklist。

**Tier**：P1（production 前必確認）
**Timing**：M1（加進部署 checklist）

---

## R23 — 無 JWT refresh token mechanism

**問題**：JWT expire = 7 天，到期 user 被踢出要 re-login。沒「靜默續期」。

**Why MVP1 acceptable**：7 天夠長給 5 人 beta。但 production SaaS 長期該補。

**修法**：引入 refresh token + rotation pattern。

**Tier**：P2
**Timing**：M2

---

## R24 — `/api/auth/login` 無 rate limit（暴力破解風險）

**問題**：[server/auth_routes.py:70-77](../../server/auth_routes.py#L70-L77) email login 無 rate limit。

**修法**：加 `slowapi` 或 `fastapi-limiter`（5 attempts/min per IP）。

**Tier**：P1（production 前必加）
**Timing**：M1

---

## R25 — bcrypt cost factor 未驗證

**問題**：[server/models.py](../../server/models.py) 用 bcrypt hash password，但沒明確設 cost factor。預設 12 可能太低（2026 標準 14+，4-5 倍時間驗證 = 防暴力破解）。

**修法**：明確設 `bcrypt.gensalt(rounds=14)`，benchmark 一次確認 < 500ms。

**Tier**：P2
**Timing**：M2

---

## R26 — useRRGData L171 故意 disable exhaustive-deps

**問題**：[useRRGData.ts:171](../../frontend/src/hooks/useRRGData.ts#L171) `// eslint-disable-line react-hooks/exhaustive-deps`，`selectedBasket` 沒列 dep（列了會無限 loop because loadRRG setSelectedBasket）。

**修法**：拆 loadRRG 變 pure（不 set state），用 React 19 `useEffectEvent` RFC pattern 分離 effect 觸發 vs 邏輯。

**Tier**：P2，**Timing**：M2，**前置**：R7 拆 useRRGData

---

## R27 — prewarm 邏輯藏在 useRRGData 內（職責散）

**問題**：[useRRGData.ts:18-49](../../frontend/src/hooks/useRRGData.ts#L18-L49) prewarm + 3 worker pool 是獨立業務邏輯，塞在 useRRGData 內讓檔案 177 行。

**修法**：抽 `usePrewarm` hook 或進 service worker / Web Worker。

**Tier**：P2，**Timing**：M2，**前置**：R7

---

## R28 — Module-level `_cache` / `_inflight` 有 HMR 議題

**問題**：[useRRGData.ts:14-16](../../frontend/src/hooks/useRRGData.ts#L14-L16) module global cache。Vite HMR 重 import 會 reset，但 already-rendered components 持有舊 reference → dev 偶爾要 hard refresh。

**修法**：TanStack Query (R15) 內建 HMR-safe cache，遷移後消滅此議題。

**Tier**：P3（dev only），**Timing**：跟 R15 一起做

---

## R29 — Frontend `ternary side-effect` anti-pattern grep audit

**問題**：`x.has(y) ? x.delete(y) : x.add(y)` 寫法散落多處。已知 2 處：
- [App.tsx:76-79](../../frontend/src/App.tsx#L76-L79) toggleSymbol
- [useFilterTags.ts:102](../../frontend/src/hooks/useFilterTags.ts#L102) toggleTag

TS 反 pattern（ternary 應該回值，不該有 side-effect）。

**修法**：全 codebase `grep -rn '\? .*delete.*: .*add'` + if/else 重寫。

**Tier**：P2，**Timing**：M2

---

## R30 — `xRef.current = x` 同步 props anti-pattern

**問題**：[useFilterTags.ts:40-41](../../frontend/src/hooks/useFilterTags.ts#L40-L41) 用 ref 補 callback closure stale。通常意味 effect/callback dep 設錯。

**修法**：React 19 `useEffectEvent` RFC 隔離；或調整 callback dep 結構。

**Tier**：P2，**Timing**：M2，**前置**：React Compiler (R18) 評估

---

## R31 — localStorage key 沒 normalize

**問題**：[useFilterTags.ts:71](../../frontend/src/hooks/useFilterTags.ts#L71) 直接用 `basketKey` 拼 LS key。未來自訂 preset name 帶特殊字元（空白 / `/` / 中文）會踩 quirks。

**修法**：抽 `lsKey(prefix, raw) → slug` 統一 normalize。

**Tier**：P3，**Timing**：等踩到

---

## R32 — `states empty` 判斷 fragile

**問題**：[useFilterTags.ts:62](../../frontend/src/hooks/useFilterTags.ts#L62) `Object.keys(states).length === 0` 判斷「沒設過」。backend 改回傳 nulls 會壞。

**修法**：HTTP 404 或 schema 加 `has_set: bool` 欄位 explicit 表達。

**Tier**：P3，**Timing**：等改 schema 時順手

---

## R33 — `filteredOutSymbols` O(N×M) perf 待驗

**問題**：[useFilterTags.ts:114-132](../../frontend/src/hooks/useFilterTags.ts#L114-L132) useMemo 在 enabled/symbols/map 任一變動重算。1000 symbol × 18 tag = 18k ops，每次 toggle 跑一次。

**修法**：incremental update（toggle 只增刪該 tag 的 symbols），或 Map 反向 index。

**Tier**：P3（1000 symbol scale 才痛），**Timing**：監控

---

## R34 — BasketDrawer 用 createPortal 取代 shadcn Sheet

**問題**：[BasketDrawer.tsx:4](../../frontend/src/components/BasketDrawer.tsx#L4) comment「specificity issues」改用 createPortal。**root cause 未文件化**。

**修法**：root cause shadcn Sheet specificity 衝突，文件化原因 + 規範後續 drawer 該用 portal 還是 Sheet。

**Tier**：P2，**Timing**：M2

---

## R35 — Confirm-Discard dialog 手刻而非 shadcn AlertDialog

**問題**：[BasketDrawer.tsx:82-101](../../frontend/src/components/BasketDrawer.tsx#L82-L101) hardcoded dialog。風險：樣式不一致 + a11y / focus trap 漏寫。

**修法**：用 shadcn AlertDialog 統一。

**Tier**：P2，**Timing**：M2

---

## R36 — 4 個 overlay 沒互斥邏輯（modal stack）

**問題**：useOverlayState 4 個 boolean（spotlight/browser/drawer/settings）可同時 true。沒 modal stack 概念。

**修法**：discriminated union `currentModal: 'spotlight' | 'browser' | 'drawer' | 'settings' | null` 強制互斥；或建 `<ModalStack>` 管理器。

**Tier**：P2，**Timing**：M2

---

## R37 — API client `headers()` / `authHeaders()` 命名不一致

**問題**：
- `tag-client.ts` `authHeaders()`
- `note-client.ts` `authHeaders()`
- `basket-client.ts` `headers()` ← 命名不同

**修法**：合 R11 一起修，統一命名 `authHeaders()`。

**Tier**：P0（跟 R11 合做）

---

## R38 — Confirm-Discard 永遠 trigger（沒偵測 dirty state）

**問題**：[BasketDrawer.tsx:33](../../frontend/src/components/BasketDrawer.tsx#L33) `tryClose` 永遠開 confirm dialog，即使 user 還沒填任何東西。

**修法**：偵測 form / AI tab dirty state，沒改動就直接 close。

**Tier**：P3，**Timing**：等用戶抱怨

---

## R39 — Spotlight / BasketBrowserPopover 邏輯重複

**問題**：兩者都是「三欄 Recent/Recommended/My + Create new」結構。

**修法**：抽 `<BasketChooser>` 共用組件，兩個 overlay 包它即可。

**Tier**：P2，**Timing**：M2，**前置**：R14 (frontend folder 分層) 一起

---

## 議題升級備註（2026-05-23 Grok 加持）

Grok outside view 給的優先順序調整：

- **R7 services.py 拆**：M2 P1 → 維持 M2 P1（backend 部分不變）
- **R7 useRRGData 拆**（split out from server-side）：M2 P1 → **P0 stop ship**（Grok 直接點名）
- **R11 authHeaders DRY**：quick win → **P0 stop ship**（在加 R15 之前必修）

理由：trading app 的資料 freshness layer 是 production blocker，加新 feature 前必須穩定。

---

## 累積規則

- 每個 review session 發現新議題 → 加新 section（下一個 R17...）
- Session 6 起草 [30-tech-roadmap.md](30-tech-roadmap.md) 時排序 + codex second opinion
- 議題開始動工 → 標 `Status: in-progress`
- 議題完成 → 標 `Status: done` + 保留歷史記錄（不刪）

## 優先序快查表（更新版，2026-05-23 Grok 加持）

| Tier | 議題 |
|------|------|
| **P0 stop ship**（推 MVP1 前必修）| R5 Redis cache / R7 useRRGData 拆 / R11 authHeaders DRY / R12 200-line hook / R15 TanStack Query / R16 Context 簡化 / R17 SymbolTable render 性能 / R18 React Compiler 評估 |
| **P1**（邊推進邊還，多項 production 前必修）| R1 server/ 分組 / R2 OpenAPI / R4 render legacy / R6 IBKR doc / R7 services.py 拆 / R9 seed_tags migration / R14 frontend folder 分層 / **R20 BASE URL 不一致** / **R22 Dev token 安全** / **R24 login rate limit** |
| **P2** | R8 globals 集中 / R10 半完成 audit / R13 過細評估 / R19 Chrome GPU 加速環境文件化 / **R21 google-auth library** / **R23 refresh token** / **R25 bcrypt cost factor** |
| **P3** | R3 split repo（never）|

## 優先序快查表（Session 6 排序前的暫定）

| Tier | 議題 |
|------|------|
| **P0** | R5 cache + Redis（production blocker）、R11 authHeaders 快勝、R12 200-line hook 放鬆 |
| **P1** | R1 server/ 結構、R2 OpenAPI 對齊、R4 render legacy、R6 IBKR doc、R7 services.py 拆、R9 seed_tags migration、R14 frontend folder 分層 |
| **P2** | R8 globals 集中、R10 半完成 audit、R13 過細評估 |
| **P3** | R3 split repo（never）|
