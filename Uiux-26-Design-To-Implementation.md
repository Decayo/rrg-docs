# Design → Implementation 對照文件

> 2026-05-07。Claude Design prototype → 實作的對應表 + 決策記錄。
> **唯一的 feature spec 文件是 doc/13**（全量 backlog）。doc/14 和 doc/22 已歸檔為歷史參考。
>
> **設計參考來源（兩種都用）：**
> - 截圖 `docs/design-prototypes/screenshots/` — 視覺 target，逼近整體感覺
> - HTML prototype `docs/design-prototypes/*.html` — 參考顏色 token、字體、組件結構、間距，但不複製 code

---

## 一、設計哲學

```
降摩擦 ←——→ 降認知    是核心取捨軸

Spotlight (⌘K)  = 摩擦最低、認知最低（快速搜尋切換）
Anchored (三欄) = 認知放這裡（分類瀏覽管理，理解全貌）
Basket Drawer   = 保持 RRG 可見，advance 操作不遮視線
PRO Lock mask   = 可複用磨砂玻璃遮罩
```

---

## 二、組件對照表

### 1. 主畫面 — Focus Layout

| 項目 | 截圖 | HTML 參考 |
|------|------|-----------|
| RRG hero（≥50% viewport），所有 panel 浮動 | `orbitflow-main-mock.png` | `OrbitFlow Mock.html` |
| Wireframe 5 方向（Focus/Workstation/Dock/iPad/K-chart） | — | `OrbitFlow Wireframes v3.html` |
| Toolbar: OrbitFlow · Discover · Watchlist · Alerts · Notes · ⌘K · avatar | 截圖頂部 | |
| Brush: 全寬 ~64px | 截圖第二行 | |
| Filter Panel: 左上 floating frosted glass | 截圖左上 | |
| Symbol Table: 左下 floating card (×/—/↗) | 截圖左下 | |
| Detail Panel: 右側 floating card (K 線 + RSI + MACD) | 截圖右側 | |

### 2. Login — Terminal Style

| 項目 | 截圖 | HTML 參考 |
|------|------|-----------|
| Terminal style C，grid 背景 | `loginv3.png` | `OrbitFlow Login v3.html` |
| email + password + Google + GitHub + dev bypass + CREATE ACCOUNT | | `OrbitFlow Login.html`（3 方向對比） |

### 3. Spotlight — ⌘K 快速切換

| 項目 | 截圖 | HTML 參考 |
|------|------|-----------|
| A2 search（單欄 command palette） | `spotlightA2.png` | `OrbitFlow Spotlight - A2 Search.html` |
| 搜尋 basket / symbol / action，BASKET/SYMBOL/ACTION 標籤 | | `OrbitFlow Spotlight - A1 Default.html` |
| 待補：QQQ\|symbolsets、tabs (ai/recent/all) | | |
| Empty state | | `OrbitFlow Spotlight - A3 Empty.html` |

### 4. Basket Browser — Anchored 三欄

| 項目 | 截圖 | HTML 參考 |
|------|------|-----------|
| B1 Anchored dropdown（從 toolbar basket title 下拉） | `B1anchor.png` | `OrbitFlow Spotlight - B1 Anchored.html` |
| 三欄：Recent / Recommended / My baskets | | `OrbitFlow Spotlight - B2 Recent.html` |
| card: emoji + name + benchmark + count + chips（單行 overflow） | | |
| 動作：+ Create new \| 🤖 AI build（並列 1:1） | | |

### 5. PRO Lock — 磨砂遮罩

| 項目 | 截圖 | HTML 參考 |
|------|------|-----------|
| A4 整個 My baskets 欄被磨砂 mask 覆蓋 | `A4 pro lock.png` | `OrbitFlow Spotlight - A4 PRO Lock.html` |
| ✦ PRO badge + Upgrade CTA，做成可複用 `<ProLockMask>` | | |

### 6. Basket Create/Edit — C Drawer

| 項目 | 截圖 | HTML 參考 |
|------|------|-----------|
| C Drawer 右側（可同時看 RRG） | `basket-c1-empty.png` | `OrbitFlow Basket - C1 Drawer Empty.html` |
| Filled: chips + valid/unknown 計數 | `basket-c2-filled.png` | `OrbitFlow Basket - C2 Drawer Filled.html` |
| Tabs: MANUAL \| AI SCREENSHOT \| IMPORT | | |
| Symbols: paste → 解析 → chips，unknown 紅框 | | |

### 7. Basket AI Screenshot

| 項目 | 截圖 | HTML 參考 |
|------|------|-----------|
| C3 AI Screenshot tab | `basket-c3-ai.png` | `OrbitFlow Basket - C3 Drawer AI.html` |
| 上傳截圖 → PARSING → 自動填 chips（chatbot 風格，最多 10 張） | | |

### 8. Badge / Pill 系統

| 項目 | 截圖 | HTML 參考 |
|------|------|-----------|
| RSI/MACD/KDJ/Pattern badge primitives | — | `OrbitFlow Components.html` |
| Trail 上的 pill（線鼓起 + badge idle/focus） | — | 同上，Section B |
| K-chart 上的 pattern tag | — | 同上，Section B |

### 9. Basket Manage（其他方向備查）

| 項目 | HTML 參考 |
|------|-----------|
| A 方向 Inline（4 狀態） | `OrbitFlow Basket - A1~A4 Inline *.html` |
| B 方向 Modal（4 狀態） | `OrbitFlow Basket - B1~B4 Modal *.html` |
| Basket 管理總覽 | `OrbitFlow Basket Manage.html` |

---

## 三、Badge / Pill / Tag 系統

> 來自 `OrbitFlow Components.html`。一套文法跨 4 個表面。

### Badge 文法

```
顏色：只有 green（漲）和 red（跌）
結構：[arrow↑] [letter] [arrow↓] [suffix]
```

| 指標 | Badge | 含義 |
|------|-------|------|
| RSI 九轉買 | `🟢 9` | bullish 9-seq |
| RSI 十三轉買 | `🟢 13` | bullish 13-seq |
| RSI 九轉賣 | `🔴 a9` | bearish 9-seq |
| RSI 十三轉賣 | `🔴 a13` | bearish 13-seq |
| MACD 零軸上金叉 | `🟢 ▲M` | strong bull continuation |
| MACD 零軸下金叉 | `🟢 ▼M` | early bull |
| MACD 零軸上死叉 | `🔴 ▲M` | early bear |
| MACD 零軸下死叉 | `🔴 ▼M` | strong bear |
| KDJ 鴨嘴向上 | `🟢 ▲K` | duckbill up |
| KDJ 鴨嘴向下 | `🔴 ▼K` | duckbill down |
| KDJ 金叉 | `🟢 K✕` | golden cross |
| KDJ 死叉 | `🔴 K✕` | death cross |
| 型態（placeholder） | `🟢 ○BH` | Bullish Harami 等 |

### 4 個表面

| 表面 | idle 狀態 | focus 狀態 |
|------|----------|-----------|
| **RRG tail** | trail 線變粗有 pill 凸起 + 小 badge | 圓點放大 + pill 放大 + 浮出詳情名稱 |
| **Symbol Table** | Patterns 欄，stacked badges（最近 5 個） | hover row → highlight |
| **K-chart Detail** | badge icon 在 candle 上方 | hover → 浮出「牛市孕育型」完整名稱 |
| **⌘K Filter** | "Pattern: 牛市孕育型 ▾" pill | 展開 dropdown |

### Trail 上的行為（重要）

```
idle:   ───●──[9]──●──[▲M]──●──  (線正常，badge 小小的貼在上面)
                                   ↑ tail 在 badge 位置稍微「鼓起」

focus:  ═══●══[🟢 9 九轉買]══●══  (線變粗，badge 放大，浮出詳情)
              ↑ 圓點變大           ↑ tooltip 顯示完整名稱+日期
```

---

## 四、影響功能的設計決策

這些 prototype 帶來的功能需求（要更新 MVP1 scope）：

| 決策 | 影響 | MVP 階段 |
|------|------|---------|
| ⌘K command palette | 新功能：全域搜尋 basket/symbol/action | M1（簡版）|
| Toolbar 重做 | OrbitFlow · Discover · Watchlist · Alerts · Notes | M1（Discover only，其他 placeholder）|
| Pattern badge 系統 | 後端需要 pattern detector（9seq/MACD/KDJ） | M2（badge UI 先做，data mock） |
| AI Screenshot basket | 後端需要 LLM 截圖解析 | M2（UI 先做，功能 mock） |
| IMPORT tab | CSV/外部匯入 | M2 |
| PRO Lock mask | 定價系統 | M2 |
| Magnetic dock | 面板拖曳 + 磁吸 + localStorage persist | M1（基礎拖曳），M2（磁吸 snap） |

---

## 五、實施順序（更新版）

```
Phase 1: Layout + 核心導航
  1. Focus layout（RRG hero + floating toolbar + brush）
  2. Basket Tabs（toolbar 裡的 pill 切換）
  3. Spotlight ⌘K（快速搜尋切換）

Phase 2: Basket 管理
  4. Anchored Browser（B1 三欄 dropdown）
  5. Basket Create Drawer（C1-C2，含 symbol multi-paste）
  6. PRO Lock mask（可複用組件）

Phase 3: Filter + Detail
  7. Filter Panel（floating frosted glass，sector + color pills）
  8. Detail Panel 重做（matching wireframe E）
  9. Badge/pill 系統（Components Canvas 的 primitives）

Phase 4: Login + 收尾
  10. Login（terminal style）
  11. 自助註冊流程（CREATE ACCOUNT）
  12. Settings
```

每個 phase 的開發流程：
```
ui-ux-designer agent → spec
  → 你確認
  → active-task → 寫 code（參照截圖，用 shadcn）
  → Chrome 截圖驗證 → checklist
```

---

## 六、HTML Prototype 參考指引

HTML prototype 的價值不只是視覺，還包含：

| 參考什麼 | 從哪裡看 | 怎麼用 |
|---------|---------|--------|
| **顏色 token** | `styles-v3.css` 的 CSS variables | 對齊到 `index.css` 的 OKLCH tokens |
| **字體** | Inter (body) + JetBrains Mono (data) + Caveat (手繪註記) | Inter 對應 system-ui，Mono 對應 tabular data |
| **間距/比例** | 各 HTML 的 padding/margin/gap 值 | 參考比例關係，不照抄 px |
| **Badge 組件** | `OrbitFlow Components.html` 的 Badge/BadgeFocus | 做 shadcn 版本時參考交互邏輯 |
| **狀態切換** | idle vs focus 的 CSS transition | 參考動畫時長和 easing |
| **frosted glass** | `backdrop-filter: blur(14px)` + oklch alpha | 統一用 CLAUDE-DESIGN.md 的規範 |

**原則：看 HTML 學設計意圖，用 shadcn 組件重新實作。**

---

## 七、文件索引

| 文件 | 角色 | 狀態 |
|------|------|------|
| **doc/13** | 全量 feature backlog（唯一維護的 feature 清單） | ✅ 活躍 |
| **本文件 (doc/26)** | design → implementation 對照 | ✅ 活躍 |
| `CLAUDE-DESIGN.md` | 設計規範（token、規則、禁止項） | ✅ 活躍 |
| `docs/25` | UI redesign spec（shadcn 組件方案） | ✅ 活躍 |
| `docs/24` | MVP1 實施計劃（W1-W6，需更新對齊 Phase） | ⚠️ 需更新 |
| `docs/design-prototypes/HANDOFF.md` | Claude Design 的 locked decisions | ✅ 參考 |
| `docs/design-prototypes/screenshots/` | 7 張設計截圖（visual target） | ✅ 參考 |
| `docs/design-prototypes/*.html` | HTML prototype（顏色/字體/組件參考） | ✅ 參考 |
| `.claude/agents/ui-ux-designer.md` | agent 定義 | ✅ 活躍 |
| `.claude/rules/ui-workflow.md` | 工作流程規則 | ✅ 活躍 |
| `docs/14` | ~~舊 basket UX spec~~ | 📦 歸檔，被 doc/26 取代 |
| `docs/22` | ~~舊 basket MVP1 spec~~ | 📦 歸檔，被 doc/24+26 取代 |
