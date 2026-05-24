# Tech Roadmap — M1 / M2 / M3

> Session 6 產出（2026-05-25）。根據 39 條 R-backlog + Codex GPT-5.5 second opinion 排序。
> Codex review 對 Grok 原始 P0 升級做了降級（讀完實際 code 後）。

## M1 — Pre-MVP1 Stop Ship ✅ COMPLETE

目標：5 人 beta 測試前必須完成。**全部完成（2026-05-25）。**

| R# | 內容 | 狀態 | 備註 |
|---|---|---|---|
| R11+R37 | authHeaders DRY — 5 client 統一 import `client.ts` | ✅ | 移除 5 份重複 |
| R12 | 200-line hook strict→warning | ✅ | 不再擋開發 |
| R20 | BASE URL default 8000→8666 + 統一 BASE export | ✅ | 6 client 統一 |
| R5 | Cache: `_mem_cache` LRU 200 上限 + parquet atomic write | ✅ | quick fix; Redis 完整遷移在 M2 |
| R17 | SymbolTable `React.memo` + `useCallback(toggleSymbol)` | ✅ | ThTip/TvButton/SymbolTable 都加 memo |
| R22+R6 | Deploy checklist + IBKR 架構約束 | ✅ | Engineering-Deploy-Checklist.md |
| R24 | Login rate limit — slowapi 5/min login, 3/min register | ✅ | 暴力破解防護 |
| R9 | seed_tags auto-seed in `init_db()` | ✅ | 新環境自動 seed |

---

## M2 — Post-MVP1 Architecture Debt

目標：MVP1 上線後，趁 codebase 還小時做架構清理。

| R# | 內容 | 狀態 | 備註 |
|---|---|---|---|
| R15 | TanStack Query — 9 hooks 遷移 | ✅ 2026-05-25 | 全 9 hooks 完成，codex review 6 issues 修完 |
| R1 | server/ flat → domain subdirs | ✅ 2026-05-25 | 29 files → 10 domain dirs |
| R17 | SymbolTable O(n²) → O(n) Map lookup | ✅ 2026-05-25 | 補上次 memo 沒做的 precompute |
| R18 | React Compiler v1.0 trial | ✅ 2026-05-25 | target:"19" config，Grok 確認 stable |
| R8 | Pydantic Settings 集中 globals | ✅ 2026-05-25 | 5 env globals 集中到 core/settings.py |
| R7 | services.py 198→153L 拆分 | ✅ 2026-05-25 | OHLCV 抽到 ohlcv.py (65L) |
| R16 | Context provider memoize | ✅ 2026-05-25 | 4 providers (Toast/Auth/Settings/Baskets) useMemo |
| R2 | OpenAPI 對齊 — response_model | ✅ 2026-05-25 | 12 endpoints + rrg router tag |
| R13 | 前端 hook/component 過細評估 — fan-in | M2 待做 | Codex audit 中 |
| R14 | frontend components/ + hooks/ 按 feature 分子資料夾 | M2 待做 | R13 先 |
| R5-full | Redis 完整遷移（response + OHLCV + data cache） | M2 待做 | 需 Redis infra |

**M2 已完成 8/11**：R1、R2、R7、R8、R15、R16、R17、R18。剩 3 項。

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

- M1：✅ 全部完成（2026-05-25）
- M2：8/11 完成（R1 / R2 / R7 / R8 / R15 / R16 / R17 / R18）— 2026-05-25
- M3：等待觸發

Last updated: 2026-05-25
