# Architecture

CartWise is a single-file application: all HTML, CSS, and JavaScript live in [`index.html`](index.html). There is **no build step, no framework, and no npm dependency**. This document explains how the code is organized so it stays easy to extend.

---

## High-level shape

```
index.html
├── <head>
│   ├── CDN links: Tabler Icons webfont + DM Sans font
│   └── <style> ............... all CSS (variables, layout, components)
└── <body>
    ├── #app
    │   ├── #sidebar .......... nav + "this month" summary panel
    │   └── #main ............. one <div class="page" id="page-X"> per screen
    ├── #modal-overlay ........ reusable modal
    ├── #toast ................ transient notifications
    └── <script> ............. all application logic
```

Every screen is a `<div class="page" id="page-X">` that is hidden by default. Navigation toggles the `.active` class to reveal exactly one page at a time.

---

## Data model

A single global object holds all state and is serialized to `localStorage` under the key `cartwise`:

```js
const STATE = {
  expenses:    [],  // grocery purchases   {id,date,item,qty,price,store,category,unit,note,meal}
  groceryList: [],  // shopping list       {id,item,qty,store,note,done}
  recurring:   [],  // recurring items     {id,item,store,category,interval,lastBought}
  budgets:     {},  // grocery budgets     { "YYYY-MM": number }
  stores:      [],  // user's store names  [ "Aldi", ... ]
  funExpenses: [],  // lifestyle spending  {id,date,what,place,category,amount,rating,note}
  wishlist:    [],  // fun wishlist        {id,what,place,category,note,done}
  funBudgets:  {}   // fun budgets         { "YYYY-MM": number }
};
```

- `load()` runs on `DOMContentLoaded`. If nothing is stored, it seeds `resetToSampleData()`.
- `save()` writes `STATE` to `localStorage` and is called after **every** mutation.

State is the single source of truth — the UI is always a pure function of `STATE`.

---

## Rendering

The UI uses a simple **rebuild-on-render** model (no virtual DOM, no reactivity library).

- Each page has a `renderX()` function that rewrites that page's `innerHTML` from `STATE`.
- A dispatcher maps page ids to their render function:

  ```js
  function render(p) {
    ({ dashboard:renderDashboard, add:renderAdd, list:renderList,
       history:renderHistory, analytics:renderAnalytics,
       recommendations:renderRecommendations, fun:renderFun,
       settings:renderSettings })[p]?.();
  }
  ```

- `goTo(pageId)` swaps the active page, updates the nav highlight, calls `render(pageId)`, and refreshes the sidebar summary.

Sub-views (e.g. the Fun page's Overview / Log / History / Wishlist tabs) follow the same pattern with their own module-level state variables (`funSubPage`, `funHistMonth`, `historyMonth`, `funRating`).

---

## Events: one delegated listener

Rather than wiring handlers to each element, interactive elements declare their intent with data attributes:

```html
<button data-action="deleteExpense" data-id="abc123">…</button>
```

A single document-level click listener routes everything:

```js
document.addEventListener('click', e => {
  const el = e.target.closest('[data-action]');
  if (!el) return;
  ACTIONS[el.dataset.action]?.(el);
});
```

All handlers live in the `ACTIONS` object and read parameters from `el.dataset` (`data-id`, `data-month`, `data-store`, `data-item`, etc.). This keeps re-rendered HTML "just works" — no need to re-bind listeners after `innerHTML` is replaced.

A few inputs that need live feedback (search box, price preview, best-price hints) attach their own `input` listeners right after their page renders.

---

## Derived data (computed, never stored)

Analytics-style values are computed on demand from `STATE.expenses`, so they're always consistent:

| Helper | Purpose |
|---|---|
| `buildRecMap()` | Groups purchases by `item → store`, computing `avgPrice`, `trend`, and the cheapest store (`__best`). Powers Best Prices, list hints, and savings. |
| `potentialSavings(monthKey)` | Sum of `(price − bestAvg) × qty` for the month — how much switching stores would save. |
| `priceAlerts()` | Flags items whose latest price at a store exceeds 115% of its historical average there. |
| `monthTotal` / `funMonthTotal` | Per-month spend rollups. |
| `recurringDueDate` / `recurringOverdue` | Drive the "due" reminders and the sidebar alert badge. |

---

## Shared UI helpers

- **Toast** — `showToast(msg)` adds `.visible` to `#toast`, auto-hiding after 2.4s.
- **Modal** — `openModal(html)` / `closeModal()`; clicking the overlay backdrop closes it.
- **Formatting** — `fmt$`, `monthKey`, `monthLabel`, `today`, `addDays`, `dateDiffDays`.
- **Colors** — `storeColor(s)` returns a fixed brand color or a stable hash-derived one; `storeTag`, `stars`, and category/icon maps keep styling consistent.
- **`esc(s)`** — HTML-escapes all user-entered text before it's inserted via `innerHTML`.

---

## Charts

All charts are dependency-free:

- **Bar / daily / month-over-month / stacked charts** are plain `<div>`s with inline `height`/`width` percentages.
- **The item price-trend chart** is hand-built inline `<svg>` (`<polyline>` per store + `<circle>` points with `<title>` tooltips).

---

## Styling

CSS is driven by custom properties on `:root` (greens for groceries, purple accents for "fun", amber highlights). Utility classes (`.card`, `.btn`, `.input`, `.badge`, `.bar-track`, etc.) are reused across pages. A single media query collapses the multi-column grids on narrow screens.

---

## Conventions when extending

1. **Mutate `STATE`, then call `save()`, then re-render** the affected page.
2. **Add new interactions as `data-action` + a handler in `ACTIONS`** — don't attach ad-hoc click listeners.
3. **Always wrap user text in `esc()`** before inserting into HTML.
4. **Keep it buildless** — no npm packages, no bundler. New third-party code must be a single CDN `<link>`/`<script>`, or it doesn't belong here (a build setup would also break the static Vercel deploy).
