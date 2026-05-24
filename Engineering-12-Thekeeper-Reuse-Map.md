# TheKeeper-ai Repos 復用地圖

> 從 TheKeeper-ai org (62 repos) 中，哪些模組可搬到 RRG 的 rrg-platform Go 服務。
> 分析日期：2026-04-30

---

## 一、Go 後端模組來源

以 `tiger-api` 為主要 clone 基礎，東補西補。

| 模組 | 主要來源 | 備用/補充 | 備註 |
|---|---|---|---|
| 服務骨架 (Gin+GORM+Redis+Zap+Viper) | `tiger-api` | `go-backend-template` | tiger-api 最完整最新 (Go 1.25) |
| JWT auth middleware | `tiger-api` | `thekeeper-api` | 兩者都用已棄用的 dgrijalva/jwt-go → 升級 golang-jwt/jwt/v5 |
| Google OAuth | `thekeeper-api` pkg/googleauth/ | — | 只有 thekeeper-api 有 |
| RBAC/Permission middleware | `tiger-api` | `thekeeper-api` | 都有 |
| Email (SMTP + 驗證信 + 重設密碼) | `modi-nfc-album-api` /email/ | — | Azure SMTP, 自製 LOGIN auth。修 InsecureSkipVerify + fmt.Println |
| Redis cache (interface + mock) | `tiger-api` pkg/cache/ | `go-backend-template` | thekeeper-api 有 Set TTL bug (傳 0) |
| PostgreSQL + migrations | `tiger-api` database/ | 所有 Go repos | golang-migrate/migrate/v4 |
| RabbitMQ producer/consumer | `tiger-api` pkg/messaging/ | `go-backend-template` | 不需要可移除 |
| Prometheus metrics | `tiger-api` middleware/ | `thekeeper-api` | promhttp handler |
| Cron scheduler | `tiger-api` | `thekeeper-api` | robfig/cron |
| Slack notify | `tiger-api` | `thekeeper-api` pkg/notify/slack/ | — |
| Helm chart (K8s) | `tiger-api` helm/ | `go-backend-template` | 改 app.name + values |
| dev docker-compose | `tiger-api` dev/ | `go-backend-template` dev/ | PG + Redis + RabbitMQ |
| Swagger | 所有 Go repos | — | swaggo/gin-swagger |

## 二、需要全新建的模組

| 模組 | 估算 | 備註 |
|---|---|---|
| Stripe Checkout + Webhooks | 2-3 天 | 無現有實作 |
| Rate limiting middleware | 0.5 天 | 用 go-redis/redis_rate |
| Token refresh endpoint | 0.5 天 | thekeeper-api 缺這塊 |
| RRG User model | 1 天 | plan tier, favorites, settings JSON |
| Filter/basket CRUD | 1 天 | 自訂 basket 存取 |

## 三、CI/CD — `thekeeper-cicd-components`

完整的 reusable GitHub Actions workflows，支援 Azure (ACR/AKS) 和 AWS (ECR/ECS)。

| Workflow | 功能 |
|---|---|
| `service_build.yml` | 版本計算 → Helm 驗證 → Docker build → ACR push → Slack |
| `service_deploy.yml` | Helm pull → 驗證 → AKS deploy → Slack |
| `service_build_aws.yml` | AWS ECR 版 build |
| `service_deploy_aws.yml` | AWS ECS 版 deploy |

接線方式：在 rrg-platform 加兩個 workflow yml 引用即可，加 `version` 檔案 (`MAJOR=1\nSPRINT=1`)。

M1 用不到（M1 的 CI 已在本 repo 的 `.github/workflows/ci.yml` 處理），M4 商業化時再接。

## 四、Infrastructure — `thekeeper-infra-kit`

Azure Terraform modules：AKS, PostgreSQL Flexible Server, Redis, Blob, ACR。
Helm 基礎設施：Prometheus, Grafana, Loki, ingress-nginx, RabbitMQ。

M1-M3 用不到（單機 docker-compose / Vercel + tunnel 夠用），M4+ 上 K8s 時參考。

## 五、前端風格對齊 — tiger-web (Andy 的標準)

RRG 維持 React + Vite SPA，不換 Next.js，但對齊慣例：

| 維度 | RRG 現狀 | tiger-web 標準 | 動作 |
|---|---|---|---|
| Prettier | ✅ 已加 (.prettierrc) | 雙引號, semi, tabWidth 2 | 已完成 |
| Tailwind plugin | ✅ 已加 | prettier-plugin-tailwindcss | 已完成 |
| ESLint + Prettier | ✅ 已加 | eslint-config-prettier | 已完成 |
| Inline styles | 🔴 SymbolTable.tsx 有殘留 | 零 inline styles | 待清理 |
| theme.ts 硬編碼色 | 🟡 #00d2ff 等 | CSS 變數 token | 待統一 |
| console.log | 🔴 可能有殘留 | loglevel 統一 logger | 待替換 |
| UI 庫 | shadcn/ui | Gluestack UI v2 | 維持 shadcn（不換） |
| State | hooks + usePersistedState | React Context + hooks | 大致一致 |

## 六、注意事項

1. **JWT 套件**：tiger-api 和 thekeeper-api 都用已棄用的 `dgrijalva/jwt-go`，新服務必須換 `golang-jwt/jwt/v5`
2. **Redis Set bug**：thekeeper-api 的 `RedisCacheService.Set()` TTL 傳了 `0`（永不過期），tiger-api 需確認
3. **Dockerfile**：所有 Go repos 都是單階段 build，建議改 multi-stage 縮小 image
4. **CORS 白名單**：thekeeper-api 硬編碼 `*.thekeeper.ai`，搬過來要改
5. **Email 模板路徑**：modi-nfc-album-api 用相對路徑 `template.ParseFiles()`，部署需注意工作目錄

## 七、如何取得 TheKeeper-ai Repos

所有 repos 皆為 private，需透過有權限的機器 clone。

**從 GitHub 直接 clone（需 TheKeeper-ai org 權限）：**
```bash
git clone git@github.com:TheKeeper-ai/tiger-api.git
git clone git@github.com:TheKeeper-ai/thekeeper-api.git
git clone git@github.com:TheKeeper-ai/modi-nfc-album-api.git
git clone git@github.com:TheKeeper-ai/thekeeper-cicd-components.git
git clone git@github.com:TheKeeper-ai/thekeeper-infra-kit.git
```

**如果 Arch 沒有 GitHub 權限，透過 SSH 從 MacBook Air 拉：**
```bash
git clone yurem@macbook-pro:/Users/yuremwu/workspace/rrg
```

MacBook Air SSH host：`macbook-pro`（Tailscale: `100.106.196.43`）
