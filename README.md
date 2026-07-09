# Bag Check — Bank of the Universe

A manual-entry personal money manager, styled like a piece of currency. One self-contained HTML file. No accounts, no bank logins, no tracking — everything lives in your browser.

*This app is for tending to all your dreams.*

---

## What it does

**The Bank** — your overview
- A "Bank of the Universe" facade with a **Total Balance** door (all your accounts, summed).
- **Money in the Bank** — what's left after your obligations and the things you've actually paid for.
- Three collapsible budget sections, each with a running subtotal:
  - **What You Owe** — bills & debts you're committed to
  - **What You Need** — essentials like food and gas
  - **What You Want** — the nice-to-haves

**The Teller** — where you manage things
- Add unlimited **bank accounts**; the app tallies your total automatically.
- Add items to any category with a name, amount, optional due day, and how often they recur.
- Filter and edit everything.

## How the money math works

- **Total Balance** = the sum of all your accounts.
- **Obligated bills (Owe)** are *always* subtracted from Money in the Bank — you're committed to them.
- **Needs & Wants** are only subtracted when you tap **Pay**. Until then they don't touch your total.
- So: **Money in the Bank = Total Balance − all Owe − the Needs/Wants you've paid.**

> Tip: treat Total Balance as your starting pot for the period (update it when money comes in), rather than lowering it every time a bill clears — otherwise a bill would get counted twice.

**Recurring:**
- Owe items can be Monthly or Yearly (yearly ones return in their anniversary month).
- Needs & Wants can be Weekly or Monthly.
- Recurring items automatically appear each new month, marked unpaid.

## Your data

Everything is saved locally in your browser via `localStorage`. Nothing is sent anywhere. Clearing your browser data (or using a different device/browser) starts fresh.

## Run it

Just open `index.html` in a browser.

### Use it like an app on your phone
1. Open the live page in Safari (or Chrome).
2. **Share → Add to Home Screen.**
3. It launches full-screen with its own icon and remembers your data.

### Host it free with GitHub Pages
1. Put these files in a repo: `index.html`, `manifest.json`, `icon-180.png`, `icon-192.png`, `icon-512.png`.
2. **Settings → Pages → Source: main / root → Save.**
3. Your app goes live at `https://YOUR-USERNAME.github.io/REPO-NAME/`.

## Tech

Vanilla HTML, CSS, and JavaScript in a single file. No build step, no dependencies, no framework. Fonts (Inter + Space Grotesk) load from Google Fonts; the guilloche rosette and bank facade are original inline SVG.

## Built by

Lain Doe · Ten Grand Company · For the 888.
