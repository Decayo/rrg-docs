# Cross-Cutting Redundancy Audit — R43-R47

> 2026-05-25 — GitNexus + grep horizontal scan after R39 BasketChooser dedup. User asked: "are there other duplicate responsibilities GitNexus could spot?"

Yes. Five distinct categories of duplication / dead code surfaced.

---

## R43 — 4 self-built createPortal modal overlays share ~50L boilerplate each

**GitNexus query**: `grep createPortal frontend/src/**/*.tsx` → 4 callers

| Component | Lines | Purpose |
|---|---|---|
| `BasketDrawer` | 102L | Right-side drawer (basket creation) |
| `ChartModal` | 50L | Fullscreen K-chart pop-out |
| `SpotlightDialog` | 72L | ⌘K basket browser (after R39 v2) |
| `BasketPanel` (pages/) | ? | Single-basket edit panel |

**Repeated boilerplate in each** (~30-50 lines per file):
```tsx
useEffect(() => {
  if (!open) return;
  const onKey = (e) => { if (e.key === "Escape") onClose(); };
  document.addEventListener("keydown", onKey);
  const prev = document.body.style.overflow;
  document.body.style.overflow = "hidden";
  return () => { /* cleanup */ };
}, [open]);

if (!open) return null;

return createPortal(
  <div className="fixed inset-0 z-50 flex items-center justify-center">
    <div className="absolute inset-0 bg-black/60" onClick={onClose} />
    <div className="relative ...inner-container...">
      {children}
    </div>
  </div>,
  document.body,
);
```

**Why 4 forks instead of using shadcn Dialog/Sheet**:
- `BasketDrawer` L4 comment claims "specificity issues" — never root-caused
- shadcn `Dialog`/`Sheet` have 0 / 3 direct callers (`ui/dialog.tsx` only used internally by sheet)
- 3 pages successfully use `Sheet` (SettingsSheet / SingleBasketEditPanel / GroupBasketViewPanel)

**Solution A** (preferred): extract `<BaseModal open onClose backdrop?>` component owning portal + body scroll lock + ESC + backdrop click. Inner content is `children`.

**Solution B**: migrate all 4 to shadcn `Dialog`/`Sheet` — but first investigate the BasketDrawer "specificity issue" claim.

**Tier**: P2 — works today, but every new modal adds 40+ duplicate lines
**Timing**: M2

---

## R44 — 5 dead shadcn primitives (0 callers)

| Primitive | Callers | Lines |
|---|---|---|
| `ui/popover.tsx` | 0 | ~85 |
| `ui/dialog.tsx` | 0 direct (sheet uses it) | ~125 |
| `ui/command.tsx` | 0 | ~157 |
| `ui/card.tsx` | 0 | ~103 |
| `ui/tabs.tsx` | 0 | ~80 |

**Why dead**:
- shadcn installs primitives via copy-paste — once added, they sit until consumed
- Some were added speculatively (BasketDrawerAITab Tab UI might have considered `tabs` then forked)
- `card` / `command` are very generic; team built domain components instead

**Recommendation**: `git rm` all 5. When a future feature needs them, run `npx shadcn@latest add <primitive>` — that's shadcn's whole design point (lib is not vendored permanently).

**Caveat**: keep `ui/dialog.tsx` because `ui/sheet.tsx` imports it. Same for any internal sibling deps. Re-verify graph before delete.

**Tier**: P3 — pure code hygiene
**Timing**: M2 — bundle size is what matters; dead primitives stay tree-shaken anyway, but file count noise is real

---

## R45 — etf/routes.py + symbol/routes.py duplicate lazy-load pattern (25L × 2)

Both files have identical structure:

```python
_DATA: ... = {}
_DATA_PATH = Path(__file__).resolve().parent.parent.parent / "data" / "X.json"

def _load_data() -> None:
    global _DATA  # noqa: PLW0603
    if _DATA: return
    if _DATA_PATH.exists():
        _DATA = json.loads(_DATA_PATH.read_text())
        logger.info("Loaded N items from %s", _DATA_PATH)
    else:
        logger.warning("X data not found: %s", _DATA_PATH)
```

**Solution**: extract `server/core/static_data.py`:
```python
def lazy_load_json(filename: str) -> dict | list:
    """Memoized JSON loader from repo-root/data/."""
    path = Path(__file__).resolve().parent.parent.parent / "data" / filename
    if not path.exists():
        logger.warning("Data not found: %s", path)
        return {} if filename.endswith(".json") else []
    return json.loads(path.read_text())
```

Or use `functools.lru_cache(maxsize=1)` per-loader.

**Tier**: P3 — works today
**Timing**: M2 — when adding 3rd static data file

---

## R46 — Loading/Empty/Error state strings scattered

**Loading**: 2 places use `<span className="text-muted-foreground animate-pulse">Loading…</span>` inline:
- `DockedChart.tsx`
- `SignalTab.tsx`

**Empty**: 4 different patterns:
- `BasketChooser`: `No results for "{query}"`
- `DetailPanel`: `No data`
- `DockedChart`: `No data (source: {source})`
- `BasketChooser` (alt): `No results`

**Recommendation**: `LoadingSkeleton.tsx` already exists as a component. Add `<EmptyState message variant="results" | "data">` and `<LoadingPulse label>` to the same module. Refactor 6+ inline JSX.

**Tier**: P3 — visual consistency wins only when more states are added
**Timing**: M2 — bundle with R43 modal extraction

---

## R47 — Backend CRUD naming inconsistency

| Domain | route function names | model function names |
|---|---|---|
| `basket` | `create_new_basket`, `update_existing_basket`, `delete_existing_basket`, `list_my_baskets` | `create_basket`, `update_basket`, `delete_basket`, `get_user_baskets` |
| `tag` | `create_new_tag`, `list_tags` | `create_tag` |
| `note` | (none with "new"/"existing") | `create_note`, `update_note`, `delete_note` |

**Issue**: `basket` and `tag` add `_new` / `_existing` to route names to disambiguate from model function names (same Python file would collide); `note` doesn't bother.

**Recommendation**: drop the `_new` / `_existing` modifiers. Routes already live in a different module (`routes.py` vs `models.py`) — no naming conflict. Adopt note's convention.

```python
# Before
@basket_router.post("")
async def create_new_basket(req: CreateBasketRequest, ...):

# After
@basket_router.post("")
async def create_basket(req: CreateBasketRequest, ...):
```

**Tier**: P3 — cosmetic, but consistency matters when reading routes side-by-side
**Timing**: M2 — quick rename pass

---

## Summary

| R# | Severity | Scope | Effort |
|---|---|---|---|
| R43 | P2 | 4 modal files + new base component | 0.5d |
| R44 | P3 | 5 dead shadcn primitives | 10min |
| R45 | P3 | 2 backend files + 1 new utility | 30min |
| R46 | P3 | 6+ inline JSX strings + LoadingSkeleton.tsx | 30min |
| R47 | P3 | ~6 function renames + their callers | 15min |

**Total quick-win batch (R44-R47)**: ~1 hour, all P3 hygiene.
**R43 modal base**: 0.5d, real architectural cleanup before adding more overlays.

Last updated: 2026-05-25
