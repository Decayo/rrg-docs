# Doc Conventions — RRG 文件撰寫規範

> Canonical 文件規範。**寫任何新 doc 前先讀這份**。
> 違反 → docs review 退回或 hook 擋下。

---

## 0. 文件層級（hierarchy）

```
L0  Entry points         README.md / CLAUDE.md / 00-doc-conventions.md
                         一張 A4 內看完，告訴新人「我是誰、規矩在哪」
L1  Section INDEX        docs/INDEX.md + docs/{section}/INDEX.md
                         地圖：列每個資料夾的子文件清單
L2  Canonical 主文       spec / roadmap / backlog / ADR / sequence
                         有編號、有更新規則、lockstep code
L3  Reference 子文       sequences/NN / adr/NNN / runbooks/NN / handoffs/NN
                         子目錄重置編號，主文的延伸
L4  Inbox 草稿           inbox/YYYY-MM-DD-{title}
                         7 天內 disposition
L-1 Archive 歷史         archive/{section}/原編號
                         不刪不改，提供脈絡
```

**規則**：
- 高層 (L0/L1) 永遠是 entry，**簡潔** — 寫太長失去 navigation 價值
- 中層 (L2/L3) 是內容主體，**詳細** — 跟 code lockstep
- 低層 (L4) 是暫存，**短命** — 7 天內處理
- 反層 (L-1) 是歷史，**不動** — git history + archive 雙保險

**讀者導引**：
- 新人來：L0 → L1 → 視需要進 L2
- 開發者改 code：L1 找對應 L2 spec，同步改
- AI agent 來：L0 自動 load + 視需要 ToolSearch L1/L2

---

## 1. 文件類型 + 各自規則

### 1.1 INDEX (`INDEX.md`)
- **每個資料夾 1 個** INDEX.md
- 列子文件清單 + 一行說明
- **加新檔同時更新對應 INDEX**（否則新檔孤兒）
- 不加編號，固定檔名 `INDEX.md`

### 1.2 ADR (Architecture Decision Record)
- **路徑**：`docs/engineering/adr/NNN-{decision-slug}.md`（**新建子資料夾**）
- **編號**：子目錄重置 sequence（001, 002, 003...）
- **必填 sections**：
  ```
  ## Status      (Proposed | Accepted | Superseded by ADR-XXX)
  ## Date        (YYYY-MM-DD)
  ## Context     (為什麼這個決策出現)
  ## Decision    (做什麼)
  ## Consequences (好處 + 壞處 + trade-off)
  ## Alternatives Considered (還有什麼選項，為什麼不選)
  ```
- **Immutable**：決策歷史，不刪不改。新決策推翻舊的另寫新 ADR + 舊的標 Superseded
- **例子**：`docs/engineering/adr/001-postgres-over-sqlite.md`

### 1.3 Sequence Diagram (User Flow)
- **路徑**：`docs/engineering/sequences/NN-{flow-name}.md`（**新建子資料夾**）
- **編號**：子目錄重置 sequence（01, 02, 03...）
- **必填 sections**：
  ```
  ## User Story  (人話描述：用戶要做什麼)
  ## Sequence Diagram (mermaid sequenceDiagram)
  ## 涉及檔案 (含 line link)
  ## 關鍵概念 (TS / React / FastAPI 概念補底)
  ## 邊界 / 已知議題 (連結到 31-refactor-backlog R#)
  ```
- 跟 code **lockstep**：改對應 code 的 PR 同步更新

### 1.4 Spec / Design Doc
- **路徑**：`docs/engineering/` (技術) / `docs/uiux/` (UI 設計) / `docs/product/` (產品需求)
- **編號**：頂層用**全域連續 sequence**（看 [INDEX.md](INDEX.md) 找下一個 free number）
- **跟 code lockstep**

### 1.5 Roadmap / Backlog
- **路徑**：`docs/product/` (feature) / `docs/engineering/` (tech)
- **rolling 文件**：持續累積，狀態改 ✅/🔲/🟡
- **不歸檔**（除非整份被取代）

### 1.6 Handoff Note
- **Review session handoff** → `~/.claude/plans/handoff-rrg-session-N.md`（**避開 docs/ hook 限制**，個人用）
- **跨人交接** → `docs/process/handoffs/NN-{topic}.md`（**新建子資料夾**）
- 一次性，做完歸檔到 `docs/archive/handoffs/`

### 1.7 Runbook / Ops
- **路徑**：`docs/process/runbooks/`（**新建子資料夾**）
- 「怎麼做 X」step-by-step
- 不會過時的操作流程

### 1.8 Inbox 草稿
- **路徑**：`docs/process/inbox/`
- **命名**：`YYYY-MM-DD-{title}.md`（時間前綴，**hook 例外**）
- **7 天內必 disposition**：融入主文 / 歸檔 / 刪
- disposition 後該檔從 inbox/ 刪除

---

## 2. 命名規範

### 2.1 強制 pattern
- 所有新 `.md` in `docs/`：**`{number}-{lowercase-title}.md`**
- 由 hook `~/.claude/hooks/guard-markdown.sh` 強制

### 2.2 允許例外
| 檔名 | 用途 |
|------|------|
| `INDEX.md` | 資料夾入口 |
| `README.md` | repo 入口 / 子目錄說明 |
| `HANDOFF.md` | 一次性交接（多在 uiux/design-prototypes 用）|
| `CHANGELOG.md` | 版本歷史 |
| `MEMORY.md` | AI 記憶 |
| `YYYY-MM-DD-{title}.md` | **限 inbox/ 內**（hook 應補例外） |

### 2.3 編號規則
- **頂層 docs/{section}/** 用**全域連續 sequence**（看歷史 + INDEX 找下一個 free）
- **子資料夾**（adr/, sequences/, handoffs/, runbooks/）用**重置 sequence**（001, 01, 02...）

---

## 3. 跨檔引用

- **一律用 markdown link**：`[title](relative-path)`
- **行號定位**：`[file.ts:42](path/to/file.ts#L42)` 或範圍 `#L42-L51`
- **❌ 不用 wiki link** `[[name]]`（Obsidian 兼容但其他工具看不懂）
- 連到 R-backlog 議題：`[R7 services.py 拆](engineering/31-refactor-backlog.md#r7-...)`

---

## 4. 長度限制

| 檔案範圍 | 上限 | 強制 |
|---------|------|------|
| `CLAUDE.md` | 80 行 | hook |
| `.claude/rules/*.md` | 100 行 | hook |
| `agent_docs/*.md` | 150 行 | hook |
| `docs/**/*.md` | 無硬性 | 超 300 行**考慮拆**（review 提醒） |

---

## 5. 文件分層（lockstep / immutable / rolling）

| 層級 | 性質 | 更新規則 | 例子 |
|------|------|---------|------|
| **lockstep** | 跟 code 同步 | 改 code 的 PR 必須同步改 docs | spec / sequence diagram / API doc |
| **immutable** | 決策歷史 snapshot | 不改，被推翻就寫新 doc + 舊的標 Superseded | ADR / handoff note |
| **rolling** | 持續累積 | 隨時 update 狀態，不歸檔（除非整份被取代） | backlog / roadmap / INDEX |
| **draft** | 暫存 | inbox/ 內 7 天內 disposition | 5/5 4 份未消化文件 |

---

## 6. 歸檔規則

- **路徑**：`docs/archive/`
- 保留**原編號**（不重編）
- 歸檔時加註「**Superseded by docs/XX-...**」在頂端
- **不刪**（git history 已記，但 archive 給後人看脈絡）
- 子資料夾歸檔（如 archive/23-new-requirement/）內檔**可不加編號**

---

## 7. Inbox 處理規則

```
草稿落入 docs/process/inbox/{YYYY-MM-DD}-{title}.md
   ↓
7 天內 disposition：
   ├─ 內容值得 → 融入相關主文（spec / ADR / backlog）+ inbox 刪
   ├─ 歷史記錄 → 歸檔到 docs/archive/inbox-history/ + inbox 刪
   └─ 沒價值 → 直接刪 + 記下為什麼刪
   ↓
inbox/ 不允許堆積超 7 天（review 時 audit）
```

---

## 8. Mermaid Diagram 規範

- **Sequence diagram**：用 `sequenceDiagram` syntax
- **Flowchart**：用 `flowchart TD`（top-down）或 `LR`（left-right）
- **State machine**：用 `stateDiagram-v2`
- **ER diagram**：用 `erDiagram`
- 範例：
  ````
  ```mermaid
  sequenceDiagram
    actor User
    participant Frontend
    participant Backend
    participant DB

    User->>Frontend: Click login
    Frontend->>Backend: POST /api/auth/login
    Backend->>DB: SELECT user
    DB-->>Backend: User row
    Backend-->>Frontend: { token, user }
    Frontend->>Frontend: setToken() to localStorage
    Frontend-->>User: Redirect to /
  ```
  ````

---

## 9. 目前已知 violations（Session 7a-c 處理）

| # | 問題 | 修法 | Session |
|---|------|------|---------|
| V1 | `TODO.md` 嚴重過時 | 重寫或刪 | 7a |
| V2 | `CLAUDE.md` 寫 `feat/frontend`（過時 branch）| 改 1 行 | 7a |
| V3 | docs/engineering/03 / 01 提 Plotly（已砍）| 刪 2-3 行 | 7a |
| V4 | inbox/ 4 份 5/5 草稿超 7 天 | disposition | 7b |
| V5 | hook `guard-markdown.sh` 無 inbox 例外 | 修 hook 邏輯 | 7c |
| V6 | INDEX.md 引用過時路徑 | 補新檔 + 刪舊 | 7c |
| V7 | docs/uiux/design-prototypes/HANDOFF.md 命名違反（沒編號）| 評估是否該編號或例外 | 7c |
| V8 | target_ui/ 子目錄內檔（01-qa, 02-...）vs 父層 25-27 編號邏輯不一致 | 確認 target_ui 是「子目錄重置 sequence」+ INDEX 補說明 | 7c |

---

## 10. 寫新 doc 前的 checklist

```
□ 確認類型（INDEX / ADR / Sequence / Spec / Roadmap / Handoff / Runbook / Inbox）
□ 看本檔對應類型的路徑 + 命名規則
□ 看頂層 docs/INDEX.md 找下一個 free 編號（如果頂層）
□ 用對應 template
□ 寫完更新對應 INDEX.md
□ 跨文件 link 用 markdown link，不用 wiki link
□ 如果是 lockstep → 對應 code PR 一起改
```

---

## 11. Living Architecture 紀律（防 over-doc）

> 來自 Grok outside view（2026-05-24）：docs-first 退化成 spec 地獄是真實風險。

### 紀律 1：高價值 docs only
- ✅ 寫：sequence diagrams / ADR / trade-off notes / refactor backlog
- ❌ 不寫：implementation detail spec（code 自己會說）、step-by-step CRUD 描述

### 紀律 2：30-60 分上限
- 每項 docs 寫**不超過 60 分**
- 超過 → 拆 / 簡化 / 改用 mermaid
- pre-MVP1 階段：**docs lead refactor 1-2 步**，再多就是負擔

### 紀律 3：寫完立刻 review/update
- code 改了 → 對應 docs **同 PR** 改
- 不立刻同步 = docs 永遠過時

### 紀律 4：AI 為主，人為輔
- AI 愛：結構化 + cross-ref + tagged sections + 明確 assumptions
- 人愛：簡潔 + 視覺化 + 故事
- **平衡**：核心可讀（人）+ 加 metadata（AI）— 例如每個 ADR 有 `## Status` `## Date` `## Decision` tags 讓 AI 解析

### Anti-pattern 警告
- ❌ 把每個 component / hook 都寫 spec（太多）
- ❌ 用 docs 替代 code comment（職責混）
- ❌ docs 寫完不維護（過時比沒有更糟）

---

## 12. Public vs Private 分流（對外分享策略）

### 12.1 公開（GitHub repo 或單獨 docs site）
- ✅ 高層 INDEX
- ✅ Roadmap（feature 列表，不含 timing / competitive）
- ✅ ADR（技術選擇 + trade-off，**非商業敏感**）
- ✅ Sequence diagrams（架構展示）
- ✅ Handoff 模板（讓未來貢獻者複用）
- ✅ Doc conventions（這份）— 示範思考深度

### 12.2 私有（.gitignore 或 separate private repo）
- ❌ Runbook（ops 細節 + 內部隱憂）
- ❌ Inbox（草稿 + idea dump）
- ❌ 商業策略 spec（定價 / KOL 名單 / 競品分析）
- ❌ **完整 trading logic / edge / alpha**（必藏，這是核心 IP）

### 12.3 分流實作（建議）
**選項 A — `.gitignore` 排除敏感 docs**：
- `docs/private/` 加 `.gitignore`，存敏感 spec
- 公開 docs 用 markdown link 引用「私有 spec 參見內部」

**選項 B — 雙 repo**：
- `rrg`（公開）只放公開 docs + code
- `rrg-private`（私有）放商業策略 + 完整 trading logic

**選項 C — Obsidian vault 分流**（推薦給單人）：
- Obsidian vault `docs/` = 公開（已是 git tracked）
- Obsidian vault `~/notes/rrg-private/` = 私有（外部，不 commit）
- 跨 vault 不能 link，但可以人工複製重點

### 12.4 對外故事框架（招 attention 用）
**HN / Dev.to / Twitter / GitHub 推銷角度**：
- 「Single dev + AI partnership build trading workstation」
- 「Docs-first vibe coding case study」
- 「Bloomberg-grade RRG for $0 self-host」
- 「Open architecture, closed alpha」

### 12.5 2026 docs 標竿參考
- [Docusaurus](https://docusaurus.io/) — docs-as-code 工具標竿
- [Linear engineering blog](https://linear.app/blog) — 極簡 decision-focused docs
- GoReleaser — docs 領先 code 演進
- Appwrite — side project 轉平台 + strong public architecture docs

---

## 13. 寫新 hook / lint rule 來強制

未來可以加：
- Hook：`docs/{section}/{number}-{title}.md` 編號連續性檢查
- Hook：INDEX.md 必須列所有同目錄 .md
- Hook：mermaid syntax 驗證
- Hook：inbox/ 7 天 disposition 提醒

目前未實作，靠 review 紀律。
