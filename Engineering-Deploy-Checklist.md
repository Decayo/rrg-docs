# Production Deployment Checklist

## Environment Variables (MUST SET)

```bash
# ❌ MUST unset in production — allows "dev-token" string to bypass auth
unset RRG_DEV_AUTH
# or explicitly: RRG_DEV_AUTH=0

# ✅ MUST set
JWT_SECRET='<generate-strong-random-64-char>'   # NOT the dev default
DATABASE_URL='postgresql://rrg:<strong-pwd>@localhost:5432/rrg'
GOOGLE_CLIENT_ID='<from-google-console>'
```

## IBKR Architecture Constraint

Backend **must** run on the same machine as IBKR TWS/Gateway.
IBKR is a desktop app binding `localhost:7497` — cannot be exposed to cloud.

```
User → Cloudflare Tunnel → Arch Linux (backend :8666) → localhost IBKR TWS
                                                      → PostgreSQL :5432
```

**Do NOT** deploy backend to cloud expecting IBKR access.
Future alternative: Alpaca / Polygon cloud API (no IBKR dependency).

## Pre-Deploy Checks

- [ ] `RRG_DEV_AUTH` unset or `0`
- [ ] `JWT_SECRET` is strong random (not dev default)
- [ ] `DATABASE_URL` points to production DB
- [ ] IBKR TWS/Gateway running on same host
- [ ] Single-worker uvicorn (multi-worker needs R5 Redis cache first)
- [ ] `ruff check` passes
- [ ] `tsc --noEmit` passes
- [ ] `vitest run` passes

## Single-Worker Enforcement (until R5 Redis done)

```bash
# ✅ Correct — single worker
uvicorn server.app:app --host 0.0.0.0 --port 8666

# ❌ WRONG — multi-worker breaks in-memory cache
uvicorn server.app:app --workers 4  # DO NOT until Redis cache (R5)
```
