# Database 決策 — 自架 PostgreSQL

> 決策日期：2026-05-03。不用 Supabase。

## 決策：自架 PostgreSQL 16（Docker Compose）

### 為什麼不用 Supabase
- 免費 tier 1 週無活動自動暫停 — 交易工具不可接受
- Supabase Auth 和自建 JWT 衝突（RLS workaround）
- Realtime 對股價 WebSocket 沒用（數據不過 DB）
- 未來遷移 graph DB 時多一個外部依賴要拆
- 500MB 限制，Notes (markdown + AI) 累積快

### 成本
- Docker self-host：$0（佔 Droplet ~100MB RAM）
- 備份：pg_dump cron → DO Spaces（~$1/月）
- 或 DO Managed Database：$15/月（自動備份 + failover）
- Beta 5-20 人：pg_dump 夠用

## Docker Compose

```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: rrg
      POSTGRES_USER: rrg_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "127.0.0.1:5432:5432"
    restart: unless-stopped

volumes:
  postgres_data:
```

## Schema 設計

### users（從 SQLite 遷移）
```sql
CREATE TABLE users (
  id          SERIAL PRIMARY KEY,
  email       TEXT UNIQUE NOT NULL,
  google_id   TEXT UNIQUE,
  password_hash TEXT,
  name        TEXT NOT NULL DEFAULT '',
  avatar_url  TEXT DEFAULT '',
  plan_tier   TEXT NOT NULL DEFAULT 'free',
  settings    JSONB NOT NULL DEFAULT '{}',
  created_at  TIMESTAMPTZ DEFAULT now(),
  updated_at  TIMESTAMPTZ DEFAULT now()
);
```

### baskets
```sql
CREATE TABLE baskets (
  id          SERIAL PRIMARY KEY,
  user_id     INTEGER NOT NULL REFERENCES users(id),
  name        TEXT NOT NULL,
  emoji       TEXT DEFAULT '',
  benchmark   TEXT NOT NULL DEFAULT 'SPY',
  symbols     TEXT[] NOT NULL DEFAULT '{}',
  created_at  TIMESTAMPTZ DEFAULT now(),
  updated_at  TIMESTAMPTZ DEFAULT now()
);
```

### notes（新增 — Obsidian-like，tag-based）
```sql
CREATE TABLE notes (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     INTEGER NOT NULL REFERENCES users(id),
  symbol      TEXT,                     -- NULL = 全域筆記
  title       TEXT,
  content     TEXT NOT NULL,            -- markdown
  tags        TEXT[] DEFAULT '{}',      -- tag-based，GIN 索引
  meta        JSONB DEFAULT '{}',       -- AI metadata、情緒分析等
  source      TEXT DEFAULT 'user',      -- 'user' | 'ai_gen' | 'pine_script'
  created_at  TIMESTAMPTZ DEFAULT now(),
  updated_at  TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_notes_user_symbol ON notes(user_id, symbol);
CREATE INDEX idx_notes_tags ON notes USING GIN(tags);
CREATE INDEX idx_notes_meta ON notes USING GIN(meta);
```

### symbol_info（i18n 全名）
```sql
CREATE TABLE symbol_info (
  ticker      TEXT PRIMARY KEY,
  name_en     TEXT NOT NULL,
  name_zh     TEXT,
  sector      TEXT,
  exchange    TEXT,
  updated_at  TIMESTAMPTZ DEFAULT now()
);
```

## Notes 設計原則

- `tags TEXT[]` + GIN index — `WHERE 'breakout' = ANY(tags)` 比 JSONB 快
- `source` 欄位區分用戶/AI/Pine Script 來源
- `meta JSONB` 存 AI 分析結果（情緒、摘要、embedding ref）
- 未來 graph DB 遷移：每筆 note = node，tags = relationships

## 未來 Graph DB

**不一定是 Neo4j**（太重、不好用）。候選：
- **Apache AGE** — PostgreSQL extension，SQL 語法查 graph，零額外 infra
- **Memgraph** — 輕量 Cypher 相容，Docker 友好
- **DuckDB** — 用戶已有經驗，OLAP 分析用

最務實路線：先用 PostgreSQL `TEXT[]` tags + `JSONB` 存關聯，
需要 graph 查詢時裝 AGE extension（不需要另一個 DB service）。

## 遷移路徑

```
Phase 1: docker-compose 加 postgres service
Phase 2: FastAPI db.py 改 asyncpg/psycopg2
Phase 3: init.sql 建表
Phase 4: SQLite 數據遷移腳本
Phase 5: 加 notes + symbol_info 表
```
