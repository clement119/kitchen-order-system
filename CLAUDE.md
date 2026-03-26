# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build step required. Open `index.html` directly in a browser or serve locally:

```bash
python -m http.server 8080
# then open http://localhost:8080
```

There are no tests, linters, or build tools configured.

## Architecture

The entire application lives in a single file: **`index.html`** (~2300 lines). It is a self-contained offline SPA with embedded CSS and JavaScript — no framework, no bundler.

### Global State

All runtime state is stored in a single global object `S`:

```js
const S = {
  view: 'summary' | 'create',
  menuItems: [],        // parsed from uploaded Excel
  orders: [],           // saved orders
  editingId: null,
  draft: { room, pax, date, servingTime, orderTime, items },
  wizard: { mealPeriod, week, day },
  drums: { h, m },
  sortOrderList: null   // SortableJS instance
}
```

`loadStorage()` / `saveStorage()` sync `S.menuItems` and `S.orders` to `localStorage` under the key `kitchen_order_app_v1`.

### Two Views

`renderContent()` dispatches to either:
- `renderSummary()` — lists all orders grouped by date/meal period; entry point for reports
- `renderCreate()` — split panel: left is the menu wizard, right is the order form

### Menu Data

Loaded from `.xlsx` via SheetJS (`libs/xlsx.min.js`). Expected columns (case-insensitive): `Meal Period`, `Week`, `Day`, `Dish / Item Name`, `Choice / Option Level 1`, `Choice / Option Level 2`.

Breakfast uses `Week=All Weeks`, `Day=Daily`. Lunch/Dinner use `Week 1–4` and `Monday–Sunday`.

### Order Data Shape

```js
{
  id: string,           // uuid()
  room: string,
  pax: number,
  date: string,         // YYYY-MM-DD
  servingTime: string,  // e.g. "Breakfast"
  orderTime: string,    // HH:MM
  items: [{ name, choice1, choice2, note, qty }]
}
```

### Third-Party Libraries (bundled in `libs/`)

| File | Library | Purpose |
|------|---------|---------|
| `xlsx.min.js` | SheetJS 0.20.3 | Excel file parsing |
| `sortable.min.js` | SortableJS 1.15.3 | Drag-to-reorder order items |
| `flatpickr.min.js/css` | Flatpickr 4.6.13 | Date picker |

### Security

User-supplied content is always passed through `escHtml()` before insertion into the DOM. All dynamic HTML generation must use this function.
