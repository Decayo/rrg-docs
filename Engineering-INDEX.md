# Engineering 🔧

> 架構、API、算法、data flow、infra 決策。
> researcher-tech agent 的主要讀取區域。

| Doc | 內容 | 狀態 |
|-----|------|------|
| [01-algorithm](01-algorithm.md) | JdK RS-Ratio/RS-Momentum 2D scatter | 穩定 |
| [03-implementation](03-implementation.md) | Python compute + 數據流 + 組件職責 | ⚠️ 嚴重過時 (R-rewrite) |
| [12-thekeeper-reuse-map](12-thekeeper-reuse-map.md) | TheKeeper-ai repos 復用分析 | 參考 |
| [17-date-architecture](17-date-architecture.md) | 日期/時區正規化 (pipeline) | 強制 |
| [21-database-decision](21-database-decision.md) | PostgreSQL 16 self-hosted (Docker) | 已定 |
| [28-data-flow-architecture](28-data-flow-architecture.md) | 時間相關開發工程標準 | 強制 |
| [30-e11-trail-signals](30-e11-trail-signals.md) | E11 Note signals 設計 | spec |
| [31-refactor-backlog](31-refactor-backlog.md) | **Canonical refactor 議題清單** (R1-R25) | rolling |

## Sequences (L3, 子目錄重置編號)

> User flow sequence diagrams (mermaid + 涉及檔案 + 議題)

| Doc | User flow | 狀態 |
|-----|-----------|------|
| [sequences/01-login-flow](sequences/01-login-flow.md) | 登入 5 entry (Google/Email/Register/Dev/Auto-verify) | lockstep |
| [sequences/02-rrg-main-flow](sequences/02-rrg-main-flow.md) | RRG 主資料流 (3 entry: cold load / warm cache / loadMore) | lockstep |
| [sequences/03-filter-tag-flow](sequences/03-filter-tag-flow.md) | Filter + Tag 系統 (3 entry: load / toggle / basket switch) | lockstep |
| [sequences/04-basket-create-flow](sequences/04-basket-create-flow.md) | Basket Create + 4 入口對比 (Drawer/Browser/Dock/Spotlight) | lockstep |
