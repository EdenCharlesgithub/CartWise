# 🛒 CartWise

A complete personal **expense tracker** for grocery spending and fun/lifestyle spending — built as a single self-contained `index.html` with **no frameworks and no build step**.

Open the file in any browser and it just works. All data is stored locally in your browser via `localStorage`.

---

## ✨ Features

### Groceries
- **Dashboard** — monthly total, end-of-month forecast, vs. last month, potential savings, budget bar with a "today" marker, daily-spend chart, category & store breakdowns, and a month-by-month grid.
- **Add Purchase** — a quick-entry parser (type `Eggs 2 Aldi 4.99`) plus a full form with live best-price hints and a running total.
- **Grocery List** — checkable list with per-item store suggestions, recurring-due reminders, and a **Trip mode** that groups items by store.
- **History** — searchable, per-month table with CSV export.
- **Analytics** — month-over-month chart, store loyalty, category breakdown, meal costs, and an SVG item price-trend chart per store.
- **Best Prices** — ranks every item's stores by average price so you always know the cheapest place to buy.

### Life & Fun
- **Fun Spending** — track dining, coffee, shopping, entertainment, travel, events, and self-care with a vibe (star) rating.
  - *Overview* — KPIs, budget bar, category breakdown, top spots, a groceries-vs-fun stacked chart, and your top-rated experiences.
  - *Log it* — quick entry form with star ratings.
  - *History* — per-month table.
  - *Wishlist* — dream experiences you want to save up for.

### Settings
- Manage your store list (with auto-assigned colors).
- Configure recurring items with preset or custom intervals.
- Set monthly budgets.
- Export to CSV, reset to sample data, or clear everything.

---

## 🚀 Getting started

### Run locally
Just open the file — no install, no server:

```
open index.html        # macOS
start index.html       # Windows
```

Or double-click `index.html` in your file explorer.

### First run
The app loads with **sample data** (April–June 2025) so every chart and feature is populated immediately. Your changes are saved automatically to `localStorage`.

> ℹ️ Because the Dashboard shows the **real** current month, its top cards start empty if today's date is past the sample data range. The History, Analytics, and Best Prices views still show the sample data. Use **Settings → Reset to sample data** at any time to restore the demo.

---

## ☁️ Deploy to Vercel

This is a static site — drop it into a repo and deploy as-is. No build command, no output directory.

```
vercel
```

The included [`vercel.json`](vercel.json) rewrites all routes to `index.html`:

```json
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```

Works the same on Netlify, GitHub Pages, or any static host.

---

## 🧰 Tech

- **Pure HTML + CSS + vanilla JavaScript** (ES6+) in one file.
- **No build step, no bundler, no npm dependencies.**
- External resources (via CDN only):
  - [Tabler Icons](https://tabler.io/icons) (webfont)
  - [DM Sans](https://fonts.google.com/specimen/DM+Sans) (Google Fonts)
- **Persistence:** `localStorage` under the key `cartwise`.

See [`ARCHITECTURE.md`](ARCHITECTURE.md) for how the code is structured.

---

## 📁 Project structure

```
.
├── index.html        # The entire app (HTML, CSS, JS)
├── vercel.json       # Static-deploy rewrites
├── README.md         # This file
└── ARCHITECTURE.md   # Code structure & conventions
```

---

## 🔒 Privacy

All data stays in your browser. Nothing is sent to any server. Clearing your browser storage (or using **Settings → Clear all data**) removes everything.
