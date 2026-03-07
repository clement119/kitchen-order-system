# Kitchen Order System

A portable, self-contained web app for composing patient meal orders and generating kitchen order reports at a post-rehab and recovery centre.

No installation, no internet connection, no build step. Open `index.html` directly in any modern browser.

---

## Quick Start

1. Open `kitchen-order/index.html` in your browser (double-click or drag into Chrome/Edge/Firefox).
2. Upload your menu via **Upload Menu** — select `Full_Menu_Consolidated.xlsx`.
3. Navigate to **Order** (+ icon in the sidebar) to create a new order.
4. Use the wizard to select Meal Period → Week → Day, then pick dishes.
5. Fill in Room, Pax, Date, and Order Time, then **Save Order**.
6. View all orders under **Summary**. Generate a **Daily Report** from there.

> To serve over localhost instead of `file://`, run:
> ```
> python -m http.server 8080 --directory kitchen-order
> ```
> Then open `http://localhost:8080`.

---

## File Structure

```
kitchen-order/
├── index.html                  # Full self-contained app (single file)
├── Full_Menu_Consolidated.xlsx # Actual facility menu (upload this)
├── libs/
│   ├── xlsx.min.js             # SheetJS 0.20.3  — Excel parsing
│   ├── sortable.min.js         # SortableJS 1.15.3 — drag-to-reorder
│   ├── flatpickr.min.js        # Flatpickr 4.6.13 — date picker
│   └── flatpickr.min.css
└── README.md
```

All state is persisted in **localStorage** (key: `kitchen_order_app_v1`). No server or database required.

---

## Menu Format

The app reads from the **`FULL MENU`** sheet of the uploaded Excel file.

| Meal Period | Week      | Day     | Dish / Item Name       | Choice / Option Level 1 | Choice / Option Level 2 |
|-------------|-----------|---------|------------------------|--------------------------|--------------------------|
| Breakfast   | All Weeks | Daily   | Porridge               | Chicken                  |                          |
| Breakfast   | All Weeks | Daily   | Porridge               | Fish                     |                          |
| Breakfast   | All Weeks | Daily   | Fried Noodle           | Spicy                    | Bee Hoon                 |
| Lunch       | Week 1    | Monday  | Sweet and Sour Chicken |                          |                          |

**Column rules:**
- `Meal Period` — `Breakfast`, `Lunch`, or `Dinner`
- `Week` — `All Weeks` (Breakfast) or `Week 1`–`Week 4` (Lunch/Dinner)
- `Day` — `Daily` (Breakfast) or `Monday`–`Sunday`
- `Dish / Item Name` — required; rows without a dish name are skipped
- `Choice / Option Level 1` — optional sub-choice (e.g. protein)
- `Choice / Option Level 2` — optional second-level choice (e.g. noodle type)

Columns are matched case-insensitively. Column order does not matter.

---

## Creating an Order

1. **Step 1 — Meal Period**: choose Breakfast, Lunch, or Dinner.
   - Breakfast skips the Week and Day steps and loads its dishes immediately.
   - Selecting a meal period also auto-sets the Serving Time field.
2. **Step 2 — Week** (Lunch/Dinner only): choose Week 1–4.
3. **Step 3 — Day** (Lunch/Dinner only): choose Monday–Sunday.
4. Dishes appear grouped by name. Click a pill to add to the order (click again to increment quantity).
5. Adjust quantity with the **−** / **+** controls on each order item.
6. Add per-item notes (e.g. "no salt", "extra portion").
7. Drag the handle on the left of any item to reorder.

**Fields:**
| Field | Notes |
|---|---|
| Room / Table Number | Free text |
| Number of Pax | How many patients share this identical order |
| Serving Time | Dropdown — auto-set by wizard, can be overridden |
| Date | Calendar picker, defaults to today |
| Order Time | Drum-scroll HH:MM picker |

---

## Reports

### Single Order
Click **View** on any order in the Summary to see a formatted text block. Use **Copy** or **Print**.

```
KITCHEN ORDER
────────────────────────────────────
Room / Table : 101
Date         : Thursday, 06 March 2026
Serving      : Breakfast
Order Time   : 07:30
Pax          : 2
────────────────────────────────────
1. Porridge (Chicken)  x2
   Note: less salt
2. Fried Noodle (Non-Spicy, Bee Hoon)
────────────────────────────────────
```

### Daily Report
Click **Daily Report** in the Summary header to generate a full-day summary grouped by meal and serve time.

```
06 Mar 2026  (Thursday)

Breakfast: (Total 3 pax)
────────────────────────────
> Serve at 07:30
* 2pax - Porridge (Chicken) x2, Fried Noodle (Non-Spicy, Bee Hoon)
         Note: less salt
* 1pax - Porridge (Fish)
```

---

## Data

- All data lives in the browser's localStorage — nothing is sent to any server.
- Clearing browser data / localStorage will erase all saved orders.
- To back up orders, use **Export** (if available) or copy the localStorage value manually.
- The menu is also cached in localStorage; re-upload if the menu file changes.
