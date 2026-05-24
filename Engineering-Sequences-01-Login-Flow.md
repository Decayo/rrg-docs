# Sequence 01 — Login Flow

> **Type**: Sequence diagram (L3, 子目錄重置編號)
> **Layer**: lockstep（跟 code 同步）
> **Last verified**: 2026-05-24 against `feat/viewport-am-fix`

## User Story

未登入 user 打開 RRG → 看到 LoginPage（3 個按鈕：Google / Email / Dev Skip）→ 選一個登入 → JWT 寫 localStorage → 跳轉 `/`（主畫面）→ 重新整理頁面，token 還在 → 自動帶 token 驗證 → 不用再登入。

涵蓋 **5 個 entry**：
1. **Google OAuth**（標準 user）
2. **Email + Password login**（已註冊 user）
3. **Email + Password register**（首次註冊）
4. **Dev bypass**（dev 環境跳過驗證）
5. **Auto-verify on mount**（reload 後恢復 session）

---

## Sequence Diagrams

### 5.1 Google OAuth Login

```mermaid
sequenceDiagram
    actor U as User
    participant LP as LoginPage.tsx
    participant Auth as useAuth (AuthProvider)
    participant Client as auth-client.ts
    participant API as auth_routes.py
    participant G as Google tokeninfo
    participant Models as models.py
    participant DB as Postgres users

    U->>LP: Click "Sign in with Google"
    LP->>LP: Google SDK 取 credential (id_token)
    LP->>Auth: login(credential)
    Auth->>Client: loginWithGoogle(credential)
    Client->>API: POST /api/auth/google { credential }
    API->>G: GET tokeninfo?id_token={credential}
    G-->>API: { sub, email, name, picture, aud, iss, email_verified }
    API->>API: 驗證 aud == GOOGLE_CLIENT_ID + iss + email_verified
    API->>Models: upsert_user(email, google_id, name, avatar_url)
    Models->>DB: INSERT or UPDATE users
    DB-->>Models: user row
    API->>API: create_access_token(user.id, user.email) → JWT (HS256, 7d)
    API-->>Client: { access_token, user }
    Client-->>Auth: AuthResponse
    Auth->>Auth: setToken(token) to localStorage
    Auth->>Auth: setUser(user)
    Auth-->>LP: Promise resolved
    LP->>LP: navigate("/")
```

### 5.2 Email + Password Login

```mermaid
sequenceDiagram
    actor U as User
    participant LP as LoginPage.tsx
    participant Auth as useAuth
    participant Client as auth-client.ts
    participant API as auth_routes.py
    participant Models as models.py
    participant DB as Postgres

    U->>LP: 填 email + password, Submit
    LP->>Auth: loginPassword(email, password)
    Auth->>Client: apiLoginPwd(email, password)
    Client->>API: POST /api/auth/login { email, password }
    API->>Models: get_user_by_email(email)
    Models->>DB: SELECT * FROM users WHERE email=$1
    DB-->>Models: user (含 password_hash) | None
    API->>API: verify_password(password, user.password_hash) via bcrypt
    alt password OK
        API->>API: create_access_token()
        API-->>Client: { access_token, user }
        Auth->>Auth: setToken + setUser
    else password 錯 / user 不存在
        API-->>Client: 401 "Invalid email or password"
        Client->>Client: throw Error
        Auth-->>LP: Promise rejected
        LP->>LP: 顯示錯誤
    end
```

### 5.3 Self-service Register

```mermaid
sequenceDiagram
    actor U as User
    participant LP as LoginPage.tsx
    participant Auth as useAuth
    participant Client as auth-client.ts
    participant API as auth_routes.py
    participant Models as models.py
    participant DB as Postgres

    U->>LP: 切到 "Create Account" tab, 填 email/password/name
    LP->>Auth: register(email, password, name)
    Auth->>Client: registerAccount(email, password, name)
    Client->>API: POST /api/auth/register { email, password, name }
    API->>Models: get_user_by_email(email)
    alt 已存在
        API-->>Client: 409 "Email already registered"
        Client->>Client: throw "Email already registered"
    else 新 user
        API->>Models: create_password_user(email, password, name)
        Models->>Models: bcrypt hash password
        Models->>DB: INSERT users (password_hash)
        DB-->>Models: user row
        API->>API: create_access_token()
        API-->>Client: 201 { access_token, user }
        Auth->>Auth: setToken + setUser
        LP->>LP: navigate("/")
    end
```

### 5.4 Dev Bypass

```mermaid
sequenceDiagram
    actor U as Dev
    participant LP as LoginPage.tsx
    participant Auth as useAuth
    participant Client as auth-client.ts
    participant API as auth_routes.py

    Note over API: 需 RRG_DEV_AUTH=1 env var
    U->>LP: Click "Skip Login (Dev)"
    LP->>Auth: loginDev()
    Auth->>Client: devLogin()
    Client->>API: POST /api/auth/dev-login (no body)
    alt RRG_DEV_AUTH 沒設
        API-->>Client: 404 "Not found"
    else dev mode
        API->>API: upsert_user(email="dev@rrg.local", ...)
        API->>API: create_access_token()
        API-->>Client: { access_token, user: "Dev User" }
        Auth->>Auth: setToken + setUser
    end
```

### 5.5 Auto-verify on Mount (reload after login)

```mermaid
sequenceDiagram
    actor U as User
    participant Main as main.tsx
    participant Auth as AuthProvider
    participant JWT as lib/jwt.ts
    participant LS as localStorage
    participant Client as auth-client.ts
    participant API as auth_routes.py

    U->>Main: 重新整理 / 首次打開
    Main->>Auth: <AuthProvider> mount
    Auth->>Auth: useEffect on mount
    Auth->>JWT: getToken()
    JWT->>LS: localStorage.getItem("rrg_token")
    LS-->>JWT: token | null
    alt token 存在
        Auth->>Client: fetchMe()
        Client->>API: GET /api/auth/me, Authorization: Bearer {token}
        API->>API: get_current_user dep 驗證 JWT
        alt JWT 有效
            API-->>Client: { user }
            Auth->>Auth: setUser(user)
        else JWT 過期/無效
            API-->>Client: 401
            Client->>Client: throw
            Auth->>JWT: removeToken()
            JWT->>LS: localStorage.removeItem
        end
    else 無 token
        Auth->>Auth: setIsLoading(false) (guest mode)
    end
```

---

## 涉及檔案

### Frontend

| 檔案 | 行 | 角色 |
|------|-----|------|
| [main.tsx](../../../frontend/src/main.tsx#L17-L42) | 17-42 | `<AuthProvider>` 包在 5 層 Provider stack 第 3 層 |
| [pages/LoginPage.tsx](../../../frontend/src/pages/LoginPage.tsx) | (143) | 3 個 button + form，呼叫 useAuth 對應 method |
| [hooks/useAuth.ts](../../../frontend/src/hooks/useAuth.ts#L23-L75) | 23-75 | Context state + 5 個 action（login/loginPassword/register/loginDev/logout）+ auto-verify useEffect |
| [api/auth-client.ts](../../../frontend/src/api/auth-client.ts) | (72) | 5 個 fetch wrapper 對應 5 個 endpoint |
| [lib/jwt.ts](../../../frontend/src/lib/jwt.ts) | (?) | getToken / setToken / removeToken (localStorage) |
| [components/AuthGuard.tsx](../../../frontend/src/components/AuthGuard.tsx) | - | 路由保護（unused now, guest mode 已開）|
| [types/auth.ts](../../../frontend/src/types/auth.ts) | - | TS 介面 User / AuthResponse / UserSettings |

### Backend

| 檔案 | 行 | 角色 |
|------|-----|------|
| [server/auth_routes.py](../../../server/auth_routes.py#L36-L125) | 36-125 | 5 個 endpoint（google/login/register/dev-login/me）+ Google token verify helper |
| [server/auth_schemas.py](../../../server/auth_schemas.py) | - | Pydantic schemas: AuthResponse / GoogleAuthRequest / LoginRequest / RegisterRequest / UserProfile |
| [server/auth_deps.py](../../../server/auth_deps.py) | (50) | `get_current_user` FastAPI Depends — Bearer token 解碼 |
| [server/jwt_utils.py](../../../server/jwt_utils.py) | (?) | create_access_token / 驗證（HS256, 7d expiry）|
| [server/models.py](../../../server/models.py) | (152) | upsert_user / create_password_user / get_user_by_email / verify_password (bcrypt) |
| [server/db.py](../../../server/db.py) | (196) | users 表 schema (R-候選: 無 alembic) |

---

## 關鍵概念補底（React / TS / FastAPI）

### React Context Provider Pattern

```typescript
// useAuth.ts L22-22
const AuthContext = createContext<AuthState | null>(null);

// L24-27: hook 是 thin wrapper，throw 防呆
export function useAuth(): AuthState {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth must be used within AuthProvider");
  return ctx;
}

// L30-75: Provider 是「狀態的擁有者」
export function AuthProvider({ children }) {
  const [user, setUser] = useState<User | null>(null);
  // ... 5 個 useCallback 包的 action
  return createElement(AuthContext.Provider, { value }, children);
}
```

**Unity 比喻**：
- `AuthContext` = `GameManager` singleton 的「插槽」
- `AuthProvider` = 真正的 `GameManager` instance + state
- `useAuth()` = `GameManager.Instance` 取得，但**保證在子樹內**（throw if outside）

### useEffect Auto-verify

```typescript
// L34-39: 只跑一次（deps = []，等同 Unity Start()）
useEffect(() => {
  const token = getToken();
  if (!token) { setIsLoading(false); return; }
  fetchMe()
    .then(setUser)
    .catch(() => removeToken())     // token 過期 → 清掉
    .finally(() => setIsLoading(false));
}, []);  // ← 空陣列 = mount-only
```

**Unity 比喻**：等同 `Start()` 跑一次，內部 await 完成才繼續。

### Discriminated Union（TS pattern 沒用到，但常見）

`AuthState` 是 plain interface，沒用 discriminated union。**未來**可改進：
```typescript
type AuthState =
  | { status: 'loading' }
  | { status: 'guest' }
  | { status: 'authenticated', user: User };
// 強制每個 branch 處理，避免 `isLoading && !user` 等 ad-hoc 組合
```

### FastAPI Dependency Injection

```python
# auth_deps.py 內 get_current_user 是 FastAPI 的 Depends
@auth_router.get("/me", response_model=UserProfile)
async def get_me(user: User = Depends(get_current_user)):  # ← inject
    return UserProfile(**user.to_dict())
```

FastAPI 自動：(1) 從 header 取 Bearer token → (2) 解 JWT → (3) DB 查 user → (4) inject into handler。如果失敗 → 自動 401，handler 不會跑。

---

## 邊界 / 已知議題（送入 [31-refactor-backlog.md](../31-refactor-backlog.md)）

### 環境配置不一致（候選 R20）
[auth-client.ts:6](../../../frontend/src/api/auth-client.ts#L6) 預設 `BASE = "http://localhost:8000"`，但 dev tmux backend 跑在 **`:8666`**。

```typescript
const BASE = import.meta.env.VITE_API_URL ?? "http://localhost:8000";
```

如果 dev 環境沒設 `VITE_API_URL`，frontend 會打 `:8000`（docker / Makefile port），而不是 tmux dev 的 `:8666`。

**修法**：把 default 改 `:8666` 或在 `frontend/.env.development` 明確設 `VITE_API_URL=http://localhost:8666`。

### `authHeaders` 在 auth-client.ts 重複實作（對應 [R11](../31-refactor-backlog.md#r11-)）
[auth-client.ts:8-11](../../../frontend/src/api/auth-client.ts#L8-L11) 自己定義 `authHeaders()`，跟 [tag-client.ts / note-client.ts / basket-client.ts](../../../frontend/src/api/) 重複。應該抽到 `api/client.ts` 統一。

### Google token 自己造輪子（候選 R21）
[auth_routes.py:97-124](../../../server/auth_routes.py#L97-L125) 用 `urllib` 打 Google tokeninfo endpoint 而非 `google-auth` 標準庫。**問題**：
- 沒 cert pinning
- 沒 retry
- 沒 cache（每次 login 都 round trip Google）

**修法**：用 `google-auth` library，內建 cache + 安全驗證。

### Dev token fallback 不安全（候選 R22）
[auth_routes.py:99](../../../server/auth_routes.py#L99) — `if _DEV_MODE and credential == "dev-token"`，任何人在 dev mode 發 `"dev-token"` 字串就能登入任何身份。**production 必須確保 `RRG_DEV_AUTH=0`**（或不設）。

CLAUDE.md / R6 (IBKR architecture constraint doc) 該明確寫「production 部署 must `RRG_DEV_AUTH` unset」。

### 無 refresh token mechanism（候選 R23）
JWT expire = 7 天，到期 user 被踢出要 re-login。沒 refresh token = 沒辦法「靜默續期」。**MVP1 acceptable**（7 天夠長）但長期該補。

### 無 email_login rate limit（候選 R24）
[auth_routes.py:70-77](../../../server/auth_routes.py#L70-L77) 沒 rate limit → 暴力破解可能。生產環境必須加（fastapi-limiter / slowapi）。

### bcrypt cost factor 未驗證（候選 R25）
[models.py](../../../server/models.py) 內 bcrypt hash 但沒指定 cost factor。預設 12 可能太低（2026 標準 14+）。要驗證。

---

## 修法優先序（送 Session 6 排序）

| 議題 | 候選 R | Tier | 緊急度 |
|------|-------|------|--------|
| BASE default port 不一致 | R20 | P1 | 馬上會踩到 |
| authHeaders DRY | 已 [R11](../31-refactor-backlog.md#r11-) | P0 | 5 分快勝 |
| Google token 標準庫 | R21 | P2 | 沒踩過事但該補 |
| Dev token 安全 | R22 | P1 | production 前必確認 |
| Refresh token | R23 | P2 | M2 補 |
| Rate limit | R24 | P1 | production 前必加 |
| bcrypt cost | R25 | P2 | M2 驗證 |

---

## Cross-references

- 上層 entry: [docs/INDEX.md](../../INDEX.md)
- Refactor backlog: [31-refactor-backlog.md](../31-refactor-backlog.md)
- Doc conventions: [00-doc-conventions.md](../../00-doc-conventions.md)
- Architecture entry: [03-implementation.md](../03-implementation.md) (⚠️ 過時)
- 後續 sequence diagrams（待寫）:
  - 02 RRG main data flow
  - 03 Filter + Tag
  - 04 Basket Create + 4 entries
