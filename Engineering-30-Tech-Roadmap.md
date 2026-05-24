# Tech Roadmap — M1 / M2 / M3

> Session 6 產出（2026-05-25）。根據 39 條 R-backlog + Codex GPT-5.5 second opinion 排序。
> Codex review 對 Grok 原始 P0 升級做了降級（讀完實際 code 後）。

## M1 — Pre-MVP1 Stop Ship

目標：5 人 beta 測試前必須完成。

### ✅ Done（本 session 已完成）

| R# | 內容 | 備註 |
|---|---|---|
| R11+R37 | authHeaders DRY — 5 client 統一 import `client.ts` | 移除 5 份重複 |
| R12 | 200-line hook strict→warning | 不再擋開發 |
| R20 | BASE URL default 8000→8666 + 統一 BASE export | note-client 8666 ↔ 其他 8000 不一致已修 |

### TODO（需用戶決策的標 ⚠️）

| R# | 內容 | 工時 | 備註 |
|---|---|---|---|
| R5 | Cache quick fix: `_mem_cache` 加 LRU 上限 + parquet atomic write | 0.5d | Codex 說 single-worker 下 P1，multi-worker 才 P0 ⚠️ |
| R17 | SymbolTable `React.memo` + `useCallback(toggleSymbol)` + precompute row map | 0.5-1d | Codex 降為 P1，但 FPS 67→10 是用戶痛感 |
| R22 | Dev token 安全 — 部署 checklist 明確寫 `RRG_DEV_AUTH` must unset | 5min | |
| R24 | Login rate limit — `slowapi` 5 attempts/min/IP | 0.5d | Production 前必加 |
| R9 | seed_tags 改 idempotent auto-seed in `init_db()` | 0.5d | 新環境不用手動跑 |
| R6 | IBKR 架構約束文件化 | 5min | 寫到 rrg-docs |

**M1 預估**：3-4 天（不含 ✅ 已完成項）

---

## M2 — Post-MVP1 Architecture Debt

目標：MVP1 上線後，趁 codebase 還小時做架構清理。

| R# | 內容 | 工時 | 依賴 | Codex 評估 |
|---|---|---|---|---|
| R15 | TanStack Query 引入 — 9 個手動 fetch hooks 遷移 | 2-4d | 無 | P1/P2，不是 stop-ship |
| R1 | server/ 按 domain 分子資料夾 (auth/ basket/ tag/ note/ ...) | 1d | 無 | |
| R7 | useRRGData (177L) + services.py (198L) 拆分 | 0.5-1d | R15 先做更好 | Codex 說 not P0 |
| R14 | frontend components/ + hooks/ 按 feature 分子資料夾 | 1d | R13 先 | |
| R16 | Context provider memoize + read/write 分離 | 0.5d | 無 | Codex 說 not P0 |
| R18 | React Compiler trial — 裝 babel plugin + lint diagnostics | 0.5-1d | R17 先 | Codex 說不能替代 R17 |
| R2 | OpenAPI 對齊 — response_model 補齊 + TS types auto-gen | 1-2d | R1 先 | |
| R8 | Pydantic Settings 集中 globals | 0.5d | R1 先 | |
| R5-full | Redis 完整遷移（response + OHLCV + data cache） | 1-2d | R5 quick fix 先 | |
| R13 | 前端 hook/component 過細評估 — fan-in 分析 | 0.5d | 無 | |

**M2 預估**：10-15 天

**建議執行順序**：
```
R15 (TanStack Query)     ← 影響最廣，先做
  → R7 (useRRGData 拆)   ← 用 useQuery 改寫時自然拆
  → R16 (Context 簡化)    ← server state 搬到 Query 後 Context 變輕
R1 (server/ 重組)         ← 跟 R15 並行
  → R8 (Pydantic Settings)
  → R2 (OpenAPI)
R13 (評估) → R14 (frontend 重組)
R18 (React Compiler trial)
R5-full (Redis)
```

---

## M3 — Future / On-Pain

等到觸發條件或用戶抱怨才做。

| R# | 內容 | 觸發條件 |
|---|---|---|
| R23 | JWT refresh token | 7 天到期用戶抱怨 |
| R21 | google-auth library 取代 urllib | 安全 audit 或多 provider |
| R25 | bcrypt cost factor 14+ | 安全 audit |
| R3 | 前後端 repo 分離 | 第二個開發者 / API 穩定 3 月 / mobile client |
| R19 | Chrome GPU 加速文件化 | 換機器 / 新開發者 |
| R26-R33 | useRRGData 內部 anti-patterns | R15 遷移時自然消滅 |
| R34-R39 | Basket/Spotlight UI 議題 | 用戶抱怨 |

---

## Codex GPT-5.5 vs Grok 評估差異

| R# | Grok 原始 | Codex 實際 | 原因 |
|---|---|---|---|
| R5 | P0 stop ship | P0 if multi-worker; P1 if single | 目前跑 single uvicorn |
| R7 | P0 (Grok 升) | Not P0 | useRRGData 177L / services 198L，沒超標 |
| R15 | P0 (Grok 升) | P1/P2 | 9 hooks 手動 fetch 但不 block MVP1 |
| R16 | P0 (Grok 升) | Not P0 | 4 providers 不多 |
| R17 | P0 | P1 | 需要先 profile 實際 jank |
| R18 | P0 | Not auto-fix | Compiler 不修 O(n²) |

**結論**：Grok 按 best practice 理論升級，Codex 讀完 code 後按實際狀態降級。真正 P0 只有 R5（multi-worker 場景）和安全項（R22/R24）。

---

## 狀態追蹤

完成項目標 ✅ + 日期。進行中標 🔄。

Last updated: 2026-05-25
