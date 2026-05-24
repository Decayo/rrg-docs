# 29 — Skills Integration: mattpocock/skills

> 整合 [mattpocock/skills](https://github.com/mattpocock/skills) 七個 skill 進 RRG 開發流程的記錄與對照
> 安裝日期：2026-05-16
> 狀態：已安裝（global `~/.claude/skills/`），觀察期
> 範圍：流程 / 對話層；**不涉及 code**

---

## 一、本次任務記錄

### 1.1 動機

對話中發現 RRG 開發有以下 friction：

- **EPIC 級任務直接開幹**：U44/U45/U46 互相依賴，但 backlog 是「事後合理化」，不是事前 grill 出來的決策樹
- **debug 缺結構**：active-task `VIEWPORT-AM-FIX` 13 edits 散在 5 個檔案，沒有明確 reproduce → hypothesise → fix loop
- **domain 詞彙分散**：`OrbitFlow / L0-L2 / tag namespace / RS-Ratio / viewport-vs-RRG-window` 散在 memory + docs/01 + docs/13 + docs/23，**無單一 glossary**
- **無 ADR**：不可逆決策（`useRTH=False`、RRG baseline 鎖 1D、lightweight-charts v5、MAX_SYMBOLS=1000）只在 commit log 裡

mattpocock/skills 主打的「對齊（grill）、精簡（domain language）、回饋迴路（diagnose/tdd）、架構（improve-codebase-architecture）」剛好對應這些 friction。

### 1.2 決策

| 項 | 結果 | 理由 |
|---|---|---|
| 安裝範圍 | 8 個（非全 14 個） | 過濾掉 GitHub-issue 相關（`to-issues`/`to-prd`/`triage`）、token 壓縮類（`caveman`）、setup 工具（`setup-matt-pocock-skills`、`write-a-skill`）、特定 codemod（`migrate-to-shoehorn` 等） |
| 安裝位置 | `~/.claude/skills/`（global） | 不限 RRG，所有專案可用；不污染 RRG repo git |
| 安裝方式 | `git clone --depth 1` + `cp -r` 手選 | 不用 `npx skills@latest add`（避免互動式 prompt 全裝、避免跑 setup script） |
| 修改既有規則 | **未動** | 等觀察期結束再決定是否改 `.claude/rules/task-discipline.md` |

### 1.3 安裝清單

| Skill | 觸發詞 | 大小 | 主要用途 |
|---|---|---|---|
| `grill-me` | 「grill me」/「stress-test this plan」 | 4 KB | 對 plan 連續質問直到 decision tree 全部解析 |
| `grill-with-docs` | 「grill with docs」 | 12 KB | 同 grill-me，但會更新 `CONTEXT.md` + 提議 ADR |
| `diagnose` | 「debug this」/「diagnose」/「something is broken」 | 12 KB | 強迫六階段：feedback loop → reproduce → hypothesise → instrument → fix → cleanup |
| `tdd` | 「TDD」/「red-green-refactor」 | 28 KB | vertical slice 紅綠燈循環，禁止 horizontal slicing |
| `improve-codebase-architecture` | 「找 deepening opportunities」/「重構」 | 20 KB | 找 shallow modules，提 deepening 候選，跑 grill loop |
| `handoff` | 「handoff to next session」 | 4 KB | 壓縮對話成 handoff doc，給下個 session/機器接手 |
| `zoom-out` | **手動** `/zoom-out` | 4 KB | 拉高層級看 module map，配 domain glossary 講話 |
| `prototype` ★ | 「prototype this」/「try a few designs」/「let me play with it」 | 12 KB | UI 分支:同路由掛多個 radical variant + bottom bar 切換;LOGIC 分支:終端 app 驗 state machine |

**總計 8 個 skill, ~96 KB**，路徑 `~/.claude/skills/<name>/SKILL.md`。
**`prototype` 是 2026-05-16 補裝**(Matt Pocock 前端圈知名度高,推薦)。

### 1.4 官方 `npx skills add` vs 本次手動安裝差異

| 面向 | 官方 | 本次 |
|---|---|---|
| 檔案內容 | 同 git repo | **100% 等效** |
| 數量 | 預設互動全裝 14 | 手選 7 |
| 安裝位置 | `~/.claude/skills/` | 同 |
| 更新 | 重跑 `npx skills@latest add` | 重 clone + cp（可包腳本） |
| setup script | 提示跑 `/setup-matt-pocock-skills` 建 issue tracker 設定 | 跳過（RRG 用 [docs/13-feature-backlog.md](13-feature-backlog.md)，不要 GitHub issues） |

**結論**：兩種方式產出檔案完全一樣，差別只在便利性與更新機制。本次手選的好處是排除跟 RRG backlog 體系衝突的 `to-issues`/`to-prd`/`triage`。

---

## 二、既有工作流（RRG 自己洗的）

### 2.1 規則檔

| 檔案 | 管轄範圍 |
|---|---|
| [CLAUDE.md](../CLAUDE.md) | 禁止事項、技術棧、品質要求 |
| [.claude/rules/task-discipline.md](../.claude/rules/task-discipline.md) | 需求 → backlog → active-task → CDD → 完成 |
| [.claude/rules/ui-workflow.md](../.claude/rules/ui-workflow.md) | UI 走 ui-ux-designer agent → spec → code → 截圖驗證 |
| [.claude/rules/ux-first.md](../.claude/rules/ux-first.md) | UX 流程先行，filter 設計參考 Notion/Airtable/Linear |
| [.claude/rules/cross-machine.md](../.claude/rules/cross-machine.md) | Arch ↔ MacBook 跨機器，tmux session |
| [docs/08-cdd-workflow.md](08-cdd-workflow.md) | Component-Driven Development 細則 |

### 2.2 流程圖

```
用戶丟需求
    │
    ▼
┌─────────────────────────────────────┐
│ 歸檔 docs/13-feature-backlog.md     │  ← 不寫 code
│ 標 priority + [M1/M2/M3]            │
└─────────────────────────────────────┘
    │
    ▼  用戶說「開始做 X」
┌─────────────────────────────────────┐
│ 建 .claude/active-task.json         │
│ {task_id, edit_count: 0, max: N}    │
└─────────────────────────────────────┘
    │
    ▼  UI 任務分叉
    ├──────────────────────────┐
    │                          │
    ▼                          ▼
ui-ux-designer agent     直接寫 code（後端/算法）
research + spec               │
    │                          │
    ▼                          │
用戶 review spec               │
    ├──────────────────────────┤
    ▼                          ▼
┌─────────────────────────────────────┐
│ 寫 code（hook 計數每次 edit）       │
│ - guard-200-lines                   │
│ - guard-no-inline-styles            │
│ - guard-active-task                 │
└─────────────────────────────────────┘
    │
    ▼  超過 max_edits → hook 暫停
    │
    ▼
CDD 驗證（Playground + Chrome 截圖 + Vitest）
    │
    ▼
刪 active-task.json，@sealed
```

### 2.3 缺口

| 缺口 | 影響 |
|---|---|
| **無 EPIC 對齊階段** | 大任務直接進 ui-ux-designer 或直接寫，依賴關係事後才浮現 |
| **無 domain glossary** | 詞彙散落，AI 每次都要重新拼湊 |
| **無 ADR** | 不可逆決策只在 commit log，未來查不到「為什麼這樣做」 |
| **debug 無結構** | 一發現 bug 就改 code，未必先建 reproduce loop |
| **TDD 不強制** | vitest 已存在但寫 test 看心情 |
| **架構 review 無入口** | 重構憑直覺，無系統化 deepening 流程 |

---

## 三、加入 skills 後的工作流

### 3.1 流程圖（**新增/變更節點以 `★` 標記**）

```
用戶丟需求
    │
    ▼
┌─────────────────────────────────────┐
│ 歸檔 docs/13-feature-backlog.md     │
└─────────────────────────────────────┘
    │
    ▼  用戶說「開始做 X」
┌─────────────────────────────────────┐
│ ★ 判斷 EPIC？                       │
│   - 小改（單檔/單組件）→ 直接做     │
│   - EPIC（多檔/多依賴）→ grill 階段 │
└─────────────────────────────────────┘
    │
    ▼  EPIC
┌─────────────────────────────────────┐
│ ★ /grill-me 或 /grill-with-docs     │
│   - 一次一題逐個釐清 decision tree  │
│   - 涉及 domain 術語 → grill-with-  │
│     docs 自動更新 CONTEXT.md / ADR  │
└─────────────────────────────────────┘
    │
    ▼  確認完整 decision tree
┌─────────────────────────────────────┐
│ 建 .claude/active-task.json         │
└─────────────────────────────────────┘
    │
    ▼  任務分叉
    ├──────────────┬──────────────┐
    │              │              │
    ▼              ▼              ▼
UI 大改       後端/算法      Debug / Perf
    │              │              │
ui-ux-designer    ★ /tdd         ★ /diagnose
    │           red-green         六階段 loop
    │              │              │
    ▼              ▼              ▼
┌─────────────────────────────────────┐
│ 寫 code（同既有 hooks）             │
│ ★ 不熟模組 → /zoom-out              │
└─────────────────────────────────────┘
    │
    ▼  寫到一半 session 太長 / 換機器
    │
    ▼  ★ /handoff → 寫 handoff doc
    │
    ▼
CDD 驗證 → 刪 active-task → @sealed
    │
    ▼  事後架構檢查（選用）
    │
    ▼  ★ /improve-codebase-architecture
       找 deepening opportunities → 進 ADR
```

### 3.2 補位對照表（**這是本次整合的核心**）

| RRG 缺口 | 補上的 skill | 怎麼補 |
|---|---|---|
| EPIC 對齊 | `grill-me` | 一次一題質問，逼回答 decision tree 每個分支 |
| domain glossary | `grill-with-docs` | 提到新術語自動 update `CONTEXT.md`，定義一句話 + 別名 |
| ADR | `grill-with-docs` | 不可逆 + 反直覺 + 真有取捨 三條件齊備才提議 ADR，存 `docs/adr/0001-*.md` |
| debug 無結構 | `diagnose` | 強迫先建 feedback loop（test/curl/script），再 hypothesise，禁未驗證假設改 code |
| TDD 不強制 | `tdd` | red→green→refactor，**vertical slice**（一 test 一 impl），禁 horizontal |
| 架構 review 無入口 | `improve-codebase-architecture` | 用 deletion test 找 shallow module，列 deepening 候選 |
| 不熟模組 | `zoom-out`（手動） | 拉高一層抽象，用 glossary 講 module map |
| 跨機器 / session 接力 | `handoff` | 寫 handoff doc 到 `mktemp`，建議下個 session 用哪些 skill |

### 3.3 跟既有規則的關係（**互補，不取代**）

| 既有規則 | skill | 關係 |
|---|---|---|
| `task-discipline.md` 的「需求 → backlog」 | `grill-me` | **強化** — backlog 之後加 grill 階段 |
| `ui-workflow.md` 的 `ui-ux-designer` agent | `grill-me` | **分流** — 小改用 grill，大改走 agent |
| `ux-first.md` 的 UX 流程先行 | `grill-with-docs` | **強化** — 用 grill 把 UX 決策樹釘死 |
| CDD（Playground + 截圖 + @sealed） | `tdd` | **互補** — CDD 管 UI 視覺，TDD 管邏輯 |
| hooks（200 行/inline style/active-task） | 全部 | **不衝突** — skill 不直接寫 code |
| `cross-machine.md`（Arch ↔ MacBook） | `handoff` | **強化** — 換機器前跑 handoff |

---

## 四、Team Orchestration Pipeline（**多 agent 協作**）

> 把 skill 從「主對話內 opt-in」升級成「orchestrator + sub-agents」協作管線。
> 前提:`~/.claude/settings.json` 已啟用 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`(✅ 已開)。

### 4.1 角色 / 模型分配

| 角色 | 模型 | 工具 | 職責 |
|---|---|---|---|
| **主 orchestrator** | opus 4.6 | 所有 skill、Agent、MCP | 對話入口、決策、Skill 觸發、agent 編排;載入 docs/13 + progress doc |
| **researcher-tech** | opus 4.6 | WebSearch、WebFetch、Bash(gh)、pal MCP (grok)、context7 | GitHub / 技術文檔 / SaaS 比對;多輪對話;每筆附連結;結束前 WebSearch 驗證 |
| **researcher-social** | opus 4.6 | WebSearch、pal MCP (grok) | X / Reddit / 社群討論 / 同類產品;多輪對話;每筆附連結;結束前 WebSearch 驗證 |
| **ui-ux-designer**(已存在) | **opus 4.7** ★ | shadcn MCP、Glob、Read、WebFetch、context7 | UI/UX 設計研究 + spec(注意:用 opus 4.7,非 4.6) |
| **coder** | opus 4.6 | 完整 file edit + Bash | 實作(套既有 hooks:200 行 / inline / active-task) |
| **codex reviewer** | GPT-5.5 high | codex CLI(via `Skill: codex` 或 tmux) | **只在 PR + debug 介入**;指出問題;不過度防禦性編程;`low` 級進 tech debt |

### 4.2 完整流程圖

```
                  ┌─────────────────────────────────┐
                  │ 用戶丟需求                       │
                  └─────────────────────────────────┘
                                  │
                                  ▼
                  ┌─────────────────────────────────┐
                  │ 主 orchestrator (opus 4.6)      │
                  │ 載入 docs/13 + 相關 progress    │
                  └─────────────────────────────────┘
                                  │
   ╔══════════════════════════════╪══════════════════════════════╗
   ║ Step 1 — 廣度搜尋(loop 入口)                                ║
   ║                              ▼                              ║
   ║  ┌──────────────────────┐ ┌──────────────────────┐         ║
   ║  │ researcher-tech       │ │ researcher-social    │         ║
   ║  │ (opus 4.6)            │ │ (opus 4.6)           │         ║
   ║  │ GitHub / 技術文檔 /   │ │ X / Reddit /         │         ║
   ║  │ SaaS                  │ │ 同類產品             │         ║
   ║  │ pal MCP (grok) 多輪 + │ │ pal MCP (grok) 多輪 +│         ║
   ║  │ 附連結 + WebSearch    │ │ 附連結 + WebSearch   │         ║
   ║  │ 驗證                  │ │ 驗證                 │         ║
   ║  └──────────────────────┘ └──────────────────────┘         ║
   ╚══════════════════════════════╪══════════════════════════════╝
                                  ▼
   ╔══════════════════════════════╪══════════════════════════════╗
   ║ Step 2 — grill                                              ║
   ║                              ▼                              ║
   ║   ┌────────────────────────────────────────────┐            ║
   ║   │ /grill-me 或 /grill-with-docs              │            ║
   ║   │ 主題:                                      │            ║
   ║   │   - 框架選擇                                │            ║
   ║   │   - 是否有更好辦法                          │            ║
   ║   │   - UI/UX 設計觀察                          │            ║
   ║   │                                            │            ║
   ║   │ 若涉及 UI/UX:                              │            ║
   ║   │ → spawn ui-ux-designer (★ opus 4.7)        │            ║
   ║   └────────────────────────────────────────────┘            ║
   ║                              │                              ║
   ║                              ▼  ★ 需要新資訊?               ║
   ║                       loop 回 Step 1                        ║
   ║                       (1→2→1, 幾輪不限)                     ║
   ╚══════════════════════════════╪══════════════════════════════╝
                                  ▼
   ╔══════════════════════════════╪══════════════════════════════╗
   ║ Step 3 — 選定搜尋(收尾確認 = 進實作前的 plan)              ║
   ║                              ▼                              ║
   ║   ┌────────────────────────────────────────────┐            ║
   ║   │ researcher(tech 或 social, 視題目)         │            ║
   ║   │ → 針對 grill 結論再驗證一次                │            ║
   ║   │ → 寫進 docs/13 / progress doc              │            ║
   ║   │   (orchestrator 之後可讀到的資訊源)        │            ║
   ║   └────────────────────────────────────────────┘            ║
   ╚══════════════════════════════╪══════════════════════════════╝
                                  ▼
                       建 .claude/active-task.json
                                  │
                                  ▼
   ╔══════════════════════════════════════════════════════════════╗
   ║ Step 4 — 實作                                                ║
   ║   ┌────────────────────────────────────────────┐            ║
   ║   │ coder (opus 4.6)                            │            ║
   ║   │ 跑 hooks: 200 行 / inline / active-task     │            ║
   ║   │                                             │            ║
   ║   │ Debug 時 → codex (GPT-5.5 high) 介入        │            ║
   ║   │   + /diagnose skill                         │            ║
   ║   └────────────────────────────────────────────┘            ║
   ╚══════════════════════════════════════════════════════════════╝
                                  │
                                  ▼
   ╔══════════════════════════════════════════════════════════════╗
   ║ Step 5 — 驗證(★ test = 未來 AI tool call 的驗證路徑)         ║
   ║   ┌────────────────────────────────────────────┐            ║
   ║   │ 排版/視覺 → Chrome 截圖(CDD)              │            ║
   ║   │ 功能/行為 → console.log + code-click       │            ║
   ║   │             button + 結果 assert            │            ║
   ║   │                                             │            ║
   ║   │ ★ 每個 test 同時是未來 AI function call    │            ║
   ║   │   的驗證路徑 — 這條一通 = function call     │            ║
   ║   │   理論上就通                                │            ║
   ║   └────────────────────────────────────────────┘            ║
   ╚══════════════════════════════════════════════════════════════╝
                                  │
                                  ▼
   ╔══════════════════════════════════════════════════════════════╗
   ║ Step 6 — PR review                                           ║
   ║   ┌────────────────────────────────────────────┐            ║
   ║   │ codex (GPT-5.5 high)                        │            ║
   ║   │ - 指出問題                                  │            ║
   ║   │ - 不過度防禦性編程                          │            ║
   ║   │ - low-level issue → tech debts / api docs   │            ║
   ║   └────────────────────────────────────────────┘            ║
   ╚══════════════════════════════════════════════════════════════╝
                                  │
                                  ▼
                       CDD @sealed + merge
```

### 4.3 設計重點

1. **researcher 兩條並行**:tech + social 平行跑,看廣度
2. **loop 1→2→1**:Step 1(廣度搜尋)→ Step 2(grill)→ 發現新需求 → 回 Step 1 補搜
3. **Step 3 是 plan 收尾**:不是再開新方向,是針對 grill 結論做最後驗證
4. **資訊持久化**:Step 3 結束時把結論寫進 docs/13 / progress doc → 主 orchestrator 之後 spawn 新 agent 都讀得到
5. **codex 只在 PR + debug 介入**:不參與 plan 階段;low-level issue 不擋 merge,進 tech debt / api docs
6. **UI/UX agent 用 opus 4.7**:模型升級理由 — UI/UX 需要更強的視覺/結構推理
7. **Test = function call 驗證路徑**:截圖看排版,但功能 test 用 console.log / code-click-button / assert 結果。**每個 test 同時是未來 AI 可調用的 tool 驗證**;test 通 = function call schema 通。這條設計打通了 RRG 「AI chatbot panel + function call」(backlog D6)的依賴鏈

### 4.4 跟現有 agents 的關係

| 現有 agent | 動作 |
|---|---|
| [`ui-ux-designer.md`](../.claude/agents/ui-ux-designer.md) | **模型升 opus 4.7**(需在 agent frontmatter 加 `model: opus`,並確認 4.7 版本) |
| (新)`researcher-tech.md` | 需新建 `.claude/agents/researcher-tech.md` |
| (新)`researcher-social.md` | 需新建 `.claude/agents/researcher-social.md` |
| (新)`coder.md` | 需新建 `.claude/agents/coder.md`;或繼續用主對話 |

### 4.5 待釐清(下一輪確認)

- [ ] researcher 兩個 agent 是否合併成一個多角色 agent(避免 spawn 開銷)
- [ ] coder 用 sub-agent vs 主對話 — 主對話 + hook 是現行,sub-agent 是 isolation 更強但 context 重建成本高
- [ ] codex 介入方式 — 用 `Skill: codex` 觸發 `codex exec` 還是另開 tmux session 並行
- [ ] tech debt 寫進哪個檔(目前無 tech debt 檔,候選:新建 `docs/31-tech-debts.md` 或加到 docs/07)
- [ ] progress doc 是 docs/13 還是新建一個(orchestrator 載入用)
- [ ] Test pipeline 細則:console.log tag、code-click-button helper、assert 規範 — 需另開 spec

---

## 五、觸發指南（快速查表）

當你說... → Claude 應該觸發...

| 用戶語句 | 觸發 |
|---|---|
| 「我想做 U44」（沒講細節） | **`grill-me`** — 不要直接寫 |
| 「對 U44 grill me」 | `grill-me` |
| 「這個跟 OrbitFlow L0/L1 怎麼擺」 | `grill-with-docs`（涉及 domain 術語） |
| 「decompose this plan」 | `grill-me` |
| 「viewport 拖到一半會 loop」 | **`diagnose`** — 不要直接改 code |
| 「RRG 載入很慢」 | `diagnose`（perf branch） |
| 「用 TDD 做 KDJ-based RRG」 | `tdd` |
| 「先寫 test 再寫 calcKDJ」 | `tdd` |
| 「frontend chart 那塊架構好複雜」 | `improve-codebase-architecture` |
| 「能不能把 DockedChart 拆得更深」 | `improve-codebase-architecture` |
| 「我不懂 lightweight-charts 怎麼運作」 | `/zoom-out`（手動） |
| 「我要去吃飯，session 給 Mac 接手」 | `handoff` |

---

## 六、不採用清單（為什麼沒裝）

| 沒裝 | 理由 |
|---|---|
| `to-issues` | RRG 用 [docs/13-feature-backlog.md](13-feature-backlog.md) 不用 GitHub issues |
| `to-prd` | 同上，PRD 在 [docs/24](24-mvp1-implementation-plan.md) 已有 |
| `triage` | 同上，分類在 backlog 表格 |
| `caveman` | 中文輸出已偏簡短，不需 token 壓縮模式 |
| `write-a-skill` | 目前不打算自寫 skill |
| `setup-matt-pocock-skills` | 跟 backlog 體系衝突 |
| `git-guardrails-claude-code` | 已有 `guard-codex-approval.sh` |
| `setup-pre-commit` | 已有 ruff + tsc + vitest pre-commit |
| `migrate-to-shoehorn` | TS 專用 codemod，不適用 |

如果未來改用 GitHub issues，把 `to-issues` + `to-prd` + `triage` 加回來就好。

---

## 七、衝突點檢查

| 風險 | 結果 |
|---|---|
| skill 修改 `settings.json` | ❌ 無 |
| skill 加 hooks | ❌ 無 |
| skill 自動寫 code | ❌ 無（全部 opt-in） |
| 200 行限制 hook 衝突 | ❌ skill 不寫 code |
| `CONTEXT.md` 已存在被覆蓋 | ⚠️ RRG 目前**無** `CONTEXT.md`，grill-with-docs 第一次跑會 lazy 建 |
| `docs/adr/` 已存在 | ⚠️ RRG 目前**無**，grill-with-docs 觸發 ADR 條件才 lazy 建 |
| 跟 `ui-ux-designer` agent 重疊 | ⚠️ 概念有重疊但走不同管道（agent 跑 background，skill 走前景對話） |

---

## 八、觀察期 + 退場機制

### 8.1 觀察期：到 2026-05-30（兩週）

判斷標準：

- [ ] 有沒有用到 `grill-me` 至少 1 次？
- [ ] 有沒有用到 `diagnose` 至少 1 次？
- [ ] `CONTEXT.md` 有沒有被建立？
- [ ] 有沒有覺得 grill 階段太煩 / 太慢？
- [ ] active-task 是否還是「edit 數爆掉但任務沒收尾」？

### 8.2 退場機制

**完全移除**：
```bash
rm -rf ~/.claude/skills/{grill-me,grill-with-docs,diagnose,tdd,improve-codebase-architecture,handoff,zoom-out,prototype}
```

**部分移除**：刪掉對應目錄即可，互相無依賴（`improve-codebase-architecture` 引用 `../grill-with-docs/*.md` 是 lazy reference，缺檔不會 crash）。

### 8.3 更新機制

需要時跑：
```bash
cd /tmp && rm -rf mp-skills && git clone --depth 1 --quiet https://github.com/mattpocock/skills.git mp-skills
for s in grill-me handoff diagnose tdd improve-codebase-architecture grill-with-docs zoom-out prototype; do
  src=$(find /tmp/mp-skills/skills -type d -name "$s" | head -1)
  [ -n "$src" ] && cp -rf "$src/." ~/.claude/skills/"$s"/
done
```

---

## 九、未動的相關問題（轉 backlog 或下次處理）

對話中浮現但**本次未處理**的問題：

1. **`VIEWPORT-AM-FIX` active-task 沒收尾**（edit_count 13/60，5 個 modified 檔案未提交）
   - 需要：CDD 驗證 + 用戶下令 commit
2. **domain glossary 缺失**（`OrbitFlow / L0-L2 / tag namespace` 散落）
   - 等第一次跑 `grill-with-docs` 自動建 `CONTEXT.md`
3. **ADR 缺失**（`useRTH=False`、RRG baseline 鎖 1D、MAX_SYMBOLS=1000 等決策無紀錄）
   - 等下次有不可逆決策時透過 `grill-with-docs` 補
4. **是否修改 `.claude/rules/task-discipline.md` 加 grill 階段**
   - 等觀察期結束（2026-05-30）再決定

---

## 十、變更紀錄

| 日期 | 動作 | 操作者 |
|---|---|---|
| 2026-05-16 | 安裝 7 個 mattpocock skill 到 `~/.claude/skills/` | Claude（用戶授權） |
| 2026-05-16 | 建立本文件 | Claude |
| 2026-05-16 | 補裝 `prototype` skill（Matt Pocock 前端圈知名度高，推薦） | Claude（用戶授權） |
| 2026-05-16 | 新增第四章 Team Orchestration Pipeline（researcher/coder/codex 多 agent 協作） | Claude（用戶授權） |
| 2026-05-30 | （預定）觀察期結束評估 | — |
