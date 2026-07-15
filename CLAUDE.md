# CLAUDE.md — Bag Check

Project context for Claude Code. Read this first.

## What this is

**Bag Check** ("Bank of the Universe") is a manual-entry personal money manager, styled like a piece of currency. It's a **single self-contained `index.html`** — vanilla HTML/CSS/JS, no build step, no dependencies, no backend. All data lives in the browser via `localStorage`. It's a Ten Grand Company product by Lain Doe.

Tagline: *"This app is for tending to all your dreams."*

## Golden rules for this repo

1. **Keep it one file.** The entire app is `index.html`. Don't split into modules or add a build system unless explicitly asked. This matches the established single-file pattern across Lain's projects.
2. **No dependencies / no framework.** Vanilla JS only. The only external load is Google Fonts (Inter + Space Grotesk). The guilloche rosette and bank facade are hand-authored inline SVG.
3. **`localStorage` must never crash the app.** All storage access is wrapped in `try/catch` (see `load()` / `save()`). This is essential — in sandboxed preview iframes `localStorage` throws, and without the guard the whole app dies. Keep this pattern for any new persisted state.
4. **Use event delegation**, not inline `onclick`. Handlers are attached to `document` (or a stable container) and match by class/data-attribute, because list HTML is re-rendered constantly. Inline handlers break on re-render.
5. **Preserve the money logic rules below exactly** — they're specific and were deliberately chosen.

## Two "pages" (tabs)

- **The Bank** (`#bank`) — the dashboard: the SVG bank facade with a single **Total Balance** door, the **Money in the Bank** vault (over a generated guilloche rosette), and three collapsible budget sections.
- **The Teller** (`#teller`) — management: add/edit accounts, add items, and a filterable "Manage Items" list.

## Data model

Everything is one object saved to `localStorage` key `bagcheck`:

```js
state = {
  accounts: [ { id, name, balance } ],              // summed for Total Balance
  currentMonth: "YYYY-MM",                           // drives month rollover
  recurringTemplates: [                              // masters for recurring items
    { id, name, amount, dueDay, category, recurrence, startMonth }
  ],
  bills: [                                           // per-month item instances
    { id, name, amount, dueDay, month, category,
      recurrence, recurring, templateId, paid, paidDate, carriedOver }
  ],
  collapsed: { owe: false, need: false, want: false }
}
```

- `category`: `'owe' | 'need' | 'want'`
- `recurrence`: `'once' | 'weekly' | 'monthly' | 'yearly'`
  - **owe** offers: once / monthly / yearly
  - **need** and **want** offer: once / weekly / monthly
- `dueDay`: `1–31` or `null` (due day is optional)
- `recurring`: boolean mirror of `recurrence !== 'once'` (kept in sync; legacy convenience)
- `carriedOver`: `true` once a bill has been rolled forward unpaid from an earlier month (see below); drives the "past due" flag in `billDueStatus()`

There is a migration block in `load()` that upgrades older saved data (single `balance` → `accounts`, missing `category`/`recurrence`/`collapsed`). Keep it.

## Money logic (do not change without being asked)

- **Total Balance** = sum of all `accounts[].balance`.
- **Money in the Bank** (`availableAfterBills()`):
  `totalBalance − (ALL owe items) − (PAID needs) − (PAID wants)`
  - **Owe items always deduct** (obligations — auto).
  - **Needs & wants deduct only when marked Paid** (tap the wax-seal "PAY").
  - Mental model for the user: Total Balance is the starting pot for the period; obligations come out automatically; tap Pay on needs/wants as you actually spend.
- **Section subtotals:**
  - **What You Owe** subtotal = sum of *all* owe items.
  - **What You Need / What You Want** subtotals = sum of *unpaid* items only (paying removes it from the "to spend" total).
- The old "OUT" door was removed on purpose (it duplicated What You Owe).

## Recurrence & month rollover

`checkMonthRollover()` runs on load. If `currentMonth` != today's `YYYY-MM`:

1. **Carry-forward pass**: any bill (any category, recurring or one-time) that's still unpaid from an earlier month has its `month` rewritten to the new current month and gets `carriedOver=true`. It is now indistinguishable from a normal this-month bill — same id, same original `dueDay`, still shows in every list, still toggles Pay/unpaid freely. Nothing ever disappears from view because of a date passing; the **only** way a bill leaves the current view is getting paid before the *next* rollover.
2. **Spawn pass**: for each `recurringTemplates` entry that should fire this month (`monthly`/`weekly` → every month; `yearly` → only on the `startMonth` anniversary), a fresh unpaid instance is created **unless** an unpaid instance for that template already exists (i.e. it just got carried forward in step 1 — don't duplicate it).

`billDueStatus(b)` computes past-due status **live**, without ever wrapping a passed due day into "next month": `carriedOver` bills are always past due (exact day-count isn't tracked across the boundary — just "past due"); same-month bills compare `dueDay` to today's day-of-month directly, so a bill due 2 days ago correctly shows "2 days past due" instead of resetting to a future date.

Known simplifications (fine for now; flag before "fixing"):
- **Weekly items count once per month** in totals (not ×4.3). A "monthly-equivalent" option is a possible feature, not a bug.
- If the app isn't opened for several months, rollover only fills the **current** month, not the skipped ones (an unpaid bill still carries forward correctly either way, since the carry-forward pass just checks `month < now`).

## Rendering

- `render()` → `renderBank()` + `renderManage()` + `renderAccounts()`.
- `renderBank()` updates the door/vault, the "Due Soon" strip (unpaid, has due date, within 7 days), and calls `renderCategory(cat, listId, subId, emptyMsg)` for owe/need/want, then `applyCollapse()`.
- `billRowHTML(b)` builds a row (wax-seal PAY/PAID toggle, name, meta line, amount). Handles `dueDay === null`.
- Segmented controls for **category** and **frequency** are built dynamically (`fillTypeSeg`) because frequency options depend on category.
- Interaction handlers are delegated on `document`: `.seal-toggle` (pay), `.mr-edit`, `.mr-del`, and the accounts list uses live `input` events (updating state without re-rendering the row, to preserve focus).

## Design system

- **Fonts:** `Inter` (body + numbers, with `tabular-nums` on money), `Space Grotesk` (wordmark + section headers). Loaded from Google Fonts.
- **CSS variables** (in `:root`): `--paper #e6e2cf`, `--paper-hi #f2efe0`, `--ink #163b2a`, `--ink-2 #274a37`, `--line #3f5c4a`, `--gold #a8863c`, `--red #8a3324`.
- **App icon / theme:** the green "B" seal; theme color `#26452e`; icon is a jewel-emerald gradient (bright top-left → deep bottom-right) with the cream seal. Files: `icon-512.png`, `icon-192.png`, `icon-180.png`, wired via `manifest.json` + `apple-touch-icon`.
- Bank facade is inline SVG (pediment, nameplate frieze "BANK OF THE UNIVERSE", fluted columns, steps). The rosette behind the vault is generated in `drawRosette()`.

## How to run

Just open `index.html` in a browser. To use on a phone: host it (GitHub Pages), open in Safari, **Share → Add to Home Screen** (full-screen + persistent, thanks to the manifest).

## How to test (recommended before committing UI/logic changes)

There's no test suite yet, but the reliable manual harness used during development is:
- `node --check` on the extracted `<script>` to catch syntax errors.
- **jsdom with `localStorage` forced to throw**, to simulate the sandbox and confirm the app still works when storage is blocked. Drive it by dispatching clicks/inputs and asserting on computed values (Total Balance, Money in the Bank, section subtotals, pay/unpay).

If you add a real test setup, keep it dependency-light (a single Node script + jsdom is plenty).

## Good first tasks / backlog ideas

- **Data export/import** (download/upload the `bagcheck` JSON) — most-requested-safety feature for a localStorage app.
- **Month history** view (past months are already stored in `bills[].month`).
- **Weekly → monthly-equivalent** toggle for budget totals.
- **Service worker** for true offline PWA.
- **Settings**: toggle whether owe deducts always vs. only-when-unpaid (some users update balances per payment).
- Optional **categories/tags** within owe/need/want.

## Files

- `index.html` — the entire app (canonical).
- `manifest.json` — PWA manifest (installable, theme color, icons).
- `icon-512.png` / `icon-192.png` / `icon-180.png` — app icons.
- `README.md` — user-facing description.
- `CLAUDE.md` — this file.
