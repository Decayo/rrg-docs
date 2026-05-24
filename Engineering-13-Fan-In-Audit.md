# Fan-In Audit — R13

> 2026-05-25 — Manual grep-based fan-in analysis. Pre-R14 decisions.

## Definition

Fan-in = number of files that reference a symbol/component name (excluding its own definition file). Low fan-in (0-2) suggests potential dead code or over-splitting candidates.

---

## Hooks (28 total)

### Dead candidates (fan-in 0)
- `useChartCrossMarkers` (64 L) — no callers, candidate for **deletion**
- `useDetailPanels` (84 L) — no callers, candidate for **deletion**
- `useRRGHistory` (51 L) — no callers, candidate for **deletion**

**Action**: verify these are truly unused (might be referenced via dynamic patterns), then delete.

### Heavy use (fan-in ≥ 4) — KEEP
- `useBaskets` (23 callers, 121 L)
- `useAuth` (18 callers, 86 L)
- `useToast` (8 callers, 44 L)
- `useSettings` (7 callers, 77 L)
- `useSymbolNames` (6 callers, 26 L)
- `useColorTags` (6 callers, 58 L)
- `useNotes` / `usePersistedState` / `useSpotlightData` / `useSymbolPaste` (4 callers each)

### Single-callsite (fan-in = 2) — candidates to inline
Most useChart* family + UI hooks:
- `useChartGMMAMarkers` (42 L), `useChartNoteLines` (94 L), `useChartOverlays` (25 L), `useChartTDMarkers` (65 L), `useChartProps` (67 L) — all single caller, family of 5 hooks → **R14 candidate: merge into `useChartMarkers`**
- `useAutoLoadMore` / `useBasketActions` / `useBrush` / `useFilterTags` / `useOverlayState` / `usePanelDrag` / `usePanelResize` / `useRRGData` / `useRRGWindow` / `useSymbolData` — kept; they encapsulate domain logic > 30 L

---

## Components (52 total — sample by fan-in)

### Suspected dead (fan-in 0) — needs verification
- `AuthGuard` (22 L)
- `BasketSelector` (27 L)
- `CreateBasketForm` (93 L)
- `FloatingPanel` (121 L)
- `OrbitPills` (64 L)
- `TopBar` (69 L)

**Note**: grep may miss aliased exports or lazy imports. Confirm before deletion.

### Merge candidates (per Codex Session 6 hint)
- `BasketDrawerAITab` + `BasketDrawerManualTab` + `BasketDrawerPlaceholderTab` — 3 tabs of one drawer, single-caller each. Merge using `<TabsContent>` prop discriminator.

---

## R14 Implications

- ~3 hooks dead → delete
- 6 components dead → verify + delete
- 5 useChart* → merge possibility
- 3 BasketDrawer*Tab → merge possibility

Final tally:
- Hooks 28 → ~24 (3 dead + 4-5 merge)
- Components 52 → ~45 (6 dead + 3 BasketDrawer merge)

These are ~20% reduction targets, not blocking but useful for R14 reorganization.

---

## Caveats

1. grep `\bname\b` counts both imports AND type references — not perfect.
2. Doesn't catch dynamic imports (`import()`, lazy()).
3. Doesn't check actual call frequency, just file-level reference.

For decisions involving deletion, manually verify each candidate.

Last updated: 2026-05-25
