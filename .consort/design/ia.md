# StockFlow Information Architecture

The screens, navigation model, and primary user flows for the StockFlow SPA.
Each screen maps to a concrete `App.tsx` route and a nav affordance; each flow
seeds the Test Strategist's E2E scenarios. Scope reflects V1: see and adjust
stock at one warehouse (file/retrieve/adjust a SKU, multi-location, receipts,
picks, single tracking code).

## Screens

- **Home — Stock by location** (`/`): the landing view. A calm, scannable
  `stock-table` inside a `card` of stock-by-location rows (SKU, location,
  quantity right-aligned in mono, stock-state `badge`). A `scan-zone` for
  barcode input at the top. Empty warehouse shows an `empty-state`.
- **SKU detail** (`/sku/:skuId`): a single narrow column. Card panels for the
  SKU's stock across locations, tracking-code detail, and stock-state. A SKU
  with no batch/serial detail shows a "not tracked" `empty-state`. Entry points
  to Adjust / Receive / Pick for this SKU.
- **Receive** (`/receive`): inbound receipt form — supplier, SKU, quantity,
  destination location. Success lands on a confirmation view; validation errors
  (unknown SKU) inline.
- **Pick** (`/pick`): outbound pick form — SKU, quantity, source location. The
  system refuses to overcommit; an overcommitting pick shows an inline error at
  the quantity field.
- **Adjust** (`/adjust`): stock-level adjustment / cycle-count form for a SKU at
  a location. Success shows an inline green flash; the stock row updates in place.
- **Search** (`/search`): find a SKU or location; results link to SKU detail.
  No matches shows an `empty-state`.

## Navigation

- **Navbar** (`navbar`, persistent): app icon + "StockFlow" wordmark (links to
  Home) on the left; nav links on the right to Home, Receive, Pick, Adjust,
  Search — the active route carries `navbar__link--active`. Every routable
  screen is reachable from here (or from a within-page link).
- **Routing** (`App.tsx` `<Routes>`): `/` Home · `/sku/:skuId` SKU detail ·
  `/receive` · `/pick` · `/adjust` · `/search`. SPA client-side routing, no
  full-page reloads. SKU detail is reached by clicking a row on Home or a Search
  result (not a top-level nav link, but reachable via those affordances).
- The barcode `scan-zone` on Home is the primary floor entry point: a scan
  resolves to a stock row update in place (success) or a persistent error toast
  (failure).

## User flows

1. **Scan-and-see (primary floor flow):** operator scans a barcode in the Home
   `scan-zone` → matching stock row flashes green and updates in place; an
   unknown/locked barcode flashes red + persistent error toast.
2. **Receive inbound:** Navbar → Receive → enter supplier, SKU, quantity,
   location → Save (brand-red primary) → confirmation view; stock at that
   location goes up.
3. **Pick outbound (no overcommit):** Navbar → Pick → enter SKU, quantity,
   source location → Save → success confirmation; a quantity beyond available
   shows an inline error at the quantity field naming it, and no stock moves.
4. **Adjust / cycle-count:** Home or SKU detail → Adjust → set new level → Save →
   inline green flash, stock row updates in place.
5. **Find a SKU:** Navbar → Search → type SKU/location → result → SKU detail; no
   match shows an empty state.
6. **Inspect a SKU:** Home row (or Search result) → SKU detail → view stock
   across locations + tracking detail; untracked SKU shows "not tracked".
