# StockFlow Design Guide

The project-level visual + interaction standards for the StockFlow SPA. This is
a CONTRACT: downstream UI is checked against it at the E2E layer. Tokens are the
source of truth in `design-guide.json`; `theme.css` is generated from it and the
component classes in `global.css` consume `var(--token)` only.

**Provenance.** BRAND + COLOR + typography from the Databricks-brand default
(`STYLE_GUIDE.md` / `theme.css`): DM Sans, navy-900 `#1B3139` text, warm-oat
`#F9F7F4` page, white cards, brand red `#FF3621` for primary/active only.
LAYOUT + information density from clean inventory dashboards observed live:
**cin7.com** (navy `rgb(33,57,76)` text on white, calm scannable tables, generous
whitespace) and **fishbowlinventory.com** (off-white `#F7F7F7` surface, dense
right-aligned numeric columns) — confirming the calm navy-on-light, high-density
table + narrow-form layout below.

## Design Philosophy

- **Calm and scannable.** The warehouse operator scans quantities and SKUs at a
  glance; the stock table is high-contrast and quiet, color is reserved for the
  primary action and for stock-state meaning.
- **One branded product.** Every screen composes the same named components; the
  warehouse app icon and "StockFlow" wordmark appear consistently.
- **Never silent.** Every state (empty, loading, success, validation error) is
  shown explicitly; every action gives feedback.
- **Floor-ready.** Large tap targets, readable at 200% zoom, barcode-scan-first.

## UI Framework and Templating

A React + TypeScript single-page app (Vite) under `client/`, client-side routed
(no full-page reloads), talking to a JSON API. Rendering stays in the boundary
layer (components/pages). Every state is a component state with a stable
`data-testid` seam and appropriate ARIA roles. No hand-assembled HTML strings.
The concrete `renders_via` framework record is the Architect's; this guide fixes
the experience: a rich client-side scan/adjust interaction, each side tested on
its own layer.

## Typography

- **UI font:** DM Sans (`--font-sans`). **Numeric/mono:** DM Mono
  (`--font-mono`) for quantity cells, with `tabular-nums` so columns align.
- **Scale:** `text-xs` 10px, `text-sm` 13px, `text-base` 15px, `text-md` 16px,
  `text-lg` 20px, `text-xl` 24px.
- **Line heights:** body 1.5, heading 1.25. **Weights:** regular 400, medium
  500, semibold 600, bold 700.

## Color Palette

- **Brand:** brand-red `#FF3621` (primary action + active state ONLY),
  brand-hover `#EB1600`, brand-light tint.
- **Semantic:** success `#2E844A`, warning `#FFAB00`, info `#0176D3`, error
  `#FF3621`, on-order `#0176D3`, quarantined `#7A4FCF` (each with a light pill
  tint). Meaning is always carried by text too, never color alone.
- **Surface:** page warm-oat `#F9F7F4`, card `#FFFFFF`, cool `#F0F2F5` (table
  header), navy-900 `#1B3139` text/dark bars, plus the navy 100–700 ramp for
  borders and secondary text.

## Spacing

4px base grid: `space-1` 4 · `space-2` 8 · `space-3` 12 · `space-4` 16 ·
`space-5` 20 · `space-6` 24 · `space-8` 32 · `space-12` 48. Generous whitespace;
the content column is centered at ~960px.

**Radius:** sharp `0` (primary CTA — a Databricks signature), `sm` 4px, `md`
8px, `lg` 12px (cards), `pill` 999px (badges).
**Shadows:** navy-tinted `sm` / `md` / `lg` for card and toast elevation.
**Breakpoints:** tablet 768px, desktop 1024px.

## Components

Every feature page COMPOSES these named classes (see `global.css`) rather than
hand-rolling markup. Vocabulary matches `design-guide.json` `components`:

- **`navbar`** — navy 64px bar, 2px brand-red bottom border; app icon +
  "StockFlow" left, nav links right, `navbar__link--active` on the current page.
- **`page`** — warm-oat bg, centered ~960px column, `page__header` +
  `page__title` (with `page__title-icon`).
- **`card`** — white surface, soft navy-tinted shadow, `--radius-lg`.
- **`btn`** — `btn--primary` (solid brand-red, sharp 0 corners), `btn--secondary`
  (outlined), `btn--ghost` (text). Tap target ≥ 44×44px.
- **`field`** — persistent visible `field__label`, `field__input` with a clear
  focus ring, `field__error` shown inline next to the offending field.
- **`stock-table`** — cool uppercase header; `stock-table__num` right-aligned
  quantity cells in mono/tabular figures.
- **`badge`** (stock-state pills) — `badge--in-stock`, `badge--low`, `badge--out`,
  `badge--on-order`, `badge--quarantined`. Meaning by text + color + shape,
  never color alone ("out" reads out AND uses error color).
- **`scan-zone`** — barcode input; `scan-zone--success` green flash (row updates
  in place), `scan-zone--error` red flash + persistent error toast.
- **`empty-state`** — icon + teaching heading + copy + CTA; also the SKU "not
  tracked" state. Never a blank region.
- **`toast`** — fixed top-right, no layout shift; `toast--ok` auto-dismisses,
  `toast--error` persists.

## Iconography

A single line-style icon set used consistently (inbound/outbound, scan,
warehouse, stock-state). Do not mix icon styles.

**App icon.** StockFlow's brand mark is the warehouse icon staged at
`.consort/design/assets/warehouse.png`, installed to `client/public/warehouse.png`.
It is REQUIRED to actually render: in the navbar next to "StockFlow"
(`navbar__icon`), in page titles (`page__title-icon`), and as the browser-tab
**favicon** wired in `index.html`. A shell shipping without the icon rendering,
or with a placeholder, has not met the brand requirement.

## User Feedback Principles

- **No silent failure, no unacknowledged success.** A successful save lands on a
  confirmation view (or an inline green flash for an adjustment); a validation
  problem (overcommitting pick, unknown SKU) shows inline next to the field that
  caused it, naming the field.
- **Scan feedback:** success = green flash + stock row updates in place; failure
  (unknown barcode, locked SKU) = scan zone flashes red + persistent toast.
- **Explicit states everywhere:** empty locations show a teaching empty state,
  untracked SKUs show "not tracked", loading and error are visible states.
- Every action surface carries a feedback affordance (`role="alert"` /
  `aria-live` region or a `data-testid` naming error/success/status).

## Accessibility

- Persistent visible labels on all inputs (not placeholder-only).
- Tap targets ≥ 44×44px on the tablet UI; keyboard-reachable; readable at 200%
  zoom (large-text default).
- Stock-state carried by shape + text, not color alone.
- Numeric quantities use tabular figures for visual column alignment.

## Adherence contract

Downstream UI is checked at the Playwright/E2E layer via
`assertDesignAdherence` (token `:root` vs this guide) plus the deterministic
element/structural checks: tokens consumed as `var(--token)` (no hardcoded
hex/px), IA `data-testid` seams present, every action gives feedback, every
feature page reachable from `App.tsx` routes + navbar, every page consumes the
component-class vocabulary. Violations are blocking `ux-adherence` smells; the UI
refactors to the guide — the guide is never weakened to match drift.
