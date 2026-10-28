# ReadyPOS: Final QA & Development Report

A record of everything I did on the `qa` branch of **ReadyPOS**, a multi-shop point-of-sale and inventory system built on Laravel: the bugs I found, how I fixed them, the features I built, and how I proved each change with an automated browser test.

| | |
|---|---|
| **Developer** | Tuhin ([@tuhin-swe](https://github.com/tuhin-swe)) |
| **Role** | QA engineer and developer: test like a customer, find the bug, fix it, prove the fix with a test |
| **Branch** | `qa` (all work merged and pushed) |
| **Stack** | Laravel 10 · PHP 8.3 · MySQL 8 · Blade · jQuery · Bootstrap 5 · ApexCharts · Spatie Permission |
| **Testing** | Playwright end-to-end tests in a real browser, on a throw-away copy of the database |

---

## At a glance

| Commits | Files touched | Lines added | Lines removed | Working days |
|:---:|:---:|:---:|:---:|:---:|
| **108** | **231** | **+21,517** | **−4,051** | **8** |

| Issues logged | Bugs fixed | Critical / major fixed | Improvements & changes | Browser tests | Recorded journeys |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **75** | **61 ** | **6 ** | **8** | **127 in 27 files** | **2 (19 steps)** |

Every issue was logged with steps to reproduce, the expected and actual result, severity, priority and how the fix was tested. The 6 bugs still open are waiting for a team decision or a test account.

### Highlights

1. **FIFO inventory costing.** Every purchase is a stock lot with its own cost and selling price. Sales take the oldest lot first, and profit comes from the real cost of the units sold.
2. **Membership points.** Customers earn points on what they buy and spend them at the POS as a discount, the way super shops run their loyalty cards. The points count as a discount everywhere in the books.
3. **Money that adds up.** Returns give back what was actually paid, discounts show in every report, and every figure was cross-checked against the database with SQL.
4. **A reporting suite the owner can trust.** Profit / Loss, Trending Products, Sales, Purchase, Expense, Stock Lots and a report per product.
5. **A faster POS.** Scanner-first cart, IMEI sales, due sales, a Sale Return button and a layout that works from phone to desktop.
6. **A complete dark theme.** Every screen I touched reads well in dark mode, from the POS buttons to the invoices, with a separate dark-mode logo.
7. **An automated safety net.** 127 browser tests and two recorded end-to-end journeys, from super admin to cashier.

![POS](images/pos.jpg)

---

## ✨ New features

### Membership points (loyalty)
- **Settings per shop:** turn points on or off, set how much spending earns a point (e.g. every 100 = 1 point), what a point is worth, and the fewest points that can be spent at once. A live example under the form shows what the rates mean.
- **At the POS:** the customer search shows each customer's points under their name. Pick a customer, press **Use points**, and the points come off the bill (never more than the customer has, or than the bill). The cashier can lower the number before paying.
- **On the invoice:** the points discount, the points earned on this sale and the new balance.
- **In the books:** the points discount is stored on the sale and counted as a discount in the Sales report, Summary report and Profit / Loss. The cash account receives only what was paid.
- **Returns, edits and deletes stay honest:** a return takes back the points its goods earned, deleting a sale gives back the points it spent, and an edited sale earns on its new total.
- **History:** a Points column on the customer list and a points history page per customer, where every change adds up to the balance.
- The server checks the balance and the minimum before anything is charged, so the screen cannot be tricked.

```mermaid
flowchart LR
    A[Sale to a saved customer<br/>pays 700] -->|every 10 = 1 point| B[+70 points]
    B --> C[Next sale: Use points<br/>40 points = 40 off]
    C --> D[Pays 260<br/>earns 26 more]
    D --> E[(Balance and history<br/>add up)]
    F[Return of goods] -->|takes back what they earned| E
```

### Point of Sale (POS)
- **Scanner-first cart.** A scanned product code, variant code or IMEI goes straight into the cart. Each variant gets its own line, capped at its own stock.
- **Redesigned product search** with keyboard-friendly results: image, code, variant, stock and price.
- **Sale Return button.** Find an invoice by typing or scanning it, return items, and go back to the POS.
- **Due sales** to saved customers, switchable in POS settings.
- **Invoice switch** in the header for the 3-inch or A4 invoice.
- **Variant-aware IMEI picking.** Choosing a variant lists only that variant's IMEIs, and the window closes cleanly with ×, Cancel or Esc.
- **Responsive layout** for phone, tablet, laptop and short or zoomed screens.

### Inventory & costing
- **FIFO stock lots.** Every purchase line is a lot with its own cost and selling price. Old stock keeps selling at its own price until it runs out, and the POS splits a cart across lots automatically (e.g. 5 × 30 + 2 × 22).
- **Returns go back to the exact lot** the units came from. Deleting or editing a sale, return or draft keeps every lot consistent.
- **Stock Lot (FIFO) report**, plus a stock report with potential profit and a stock history per product.

### IMEI / serial tracking
- Every phone is traceable from purchase to sale to return, with the variant detected automatically.
- Search or scan an IMEI in the POS to sell that exact unit.
- **Purchase returns by IMEI.** Tick the exact units going back to the supplier; those units leave stock as "returned to supplier".
- **IMEI / Serial report** with status, supplier, customer and a history timeline per unit.

### Reports
- **Profit / Loss statement.** Net sales → cost of goods (FIFO) → gross profit → expenses → net profit, each with its formula; profit by product, category, brand, warehouse, invoice, date, customer, weekday and seller.
- **Trending Products**, **Sales Report**, **one Purchase Report** (by invoice or by product), **a report per product**, **Expense Report**.
- **One date filter on every dated list and report**, with "All time" as the first choice inside the picker.

### Sales, purchases & accounts
- **Edit and delete a sale.** Stock, points, wallet and cash are reversed in one transaction.
- **Return with exchange.** Only the difference changes hands.
- **Faster sale returns.** Pick a recent invoice from a list, every line starts at 0, and the refund fills itself in (less any due it clears first). Making a return smaller lowers the refund with it.
- **Purchases.** Selling price per line, all variants at once, a printable A4 purchase invoice, and Purchase Return from the list.
- **POS cash account** with withdraw, transfer and a full history; **ledgers** with collect / pay and statements.

### Settings & everyday usability
- **Decimal places.** The shop types how many digits money shows after the point (0 to 6). Lists, POS, reports, invoices and labels all follow it; amounts are still calculated and saved in full.
- **A logo for each theme.** Separate light-mode and dark-mode logos. The dark one shows in the sidebar, the POS and the detail windows in dark mode; printing always uses the light one.
- **See what is uploaded.** Each logo field and the profile photo show the image in use, and a newly picked file before it is saved.
- **Invoice Settings** with a live preview; **roles that mean it** in the admin panel, the POS and the mobile API; a **notification bell** for new sales and low stock.
- **No more cut-off names**, **simpler supplier and customer forms**, and **one selling price everywhere**.

### Removed at the team's request
- **Coupons.** The discount at the till now comes from membership points. The Coupon pages, the POS coupon box and their permissions are gone; past sales that used a coupon keep their discount in invoices and reports.
- **Settings > License.** The page and its permissions are gone; the installer's licence step is kept.

---

## 🐞 Bugs fixed (highlights)

### Security & data integrity
- **Multi-shop isolation** applied to every authenticated mobile API route, plus subscription checks the same as on the web.
- **Hardened password reset** in the mobile app, a **safer database backup**, and a failed mobile-money payment no longer completes a sale.

### Money & stock correctness
- **Returns refunded more than was paid.** A sale with a discount, coupon or points was returned at the full price, so the customer got back more than they paid and Profit / Loss took too much off sales. Returns are now worth what was paid for the goods.
- **Discounts were invisible.** The Sales report showed $0 discount, and the Summary report always showed 0.
- **Profit / Loss** ignored the quantity sold and showed sales as "cost of goods".
- **Deleting a used expense category** crashed with a database error; now it is kept and the page says how many expenses use it.
- **Making a sale return smaller** could not be saved because the old refund was now too high.
- **Adjust Stock** on a stock count did not change the stock; **a due sale collected in full** stayed due.
- **Editing a purchase** reset line costs; **return edits** moved stock the wrong way; **returns** mixed up two lines of the same product.
- **Deleting a purchase that had a return** took the returned units out of stock twice; **returned phones** stayed sellable.

### POS & screens
- **Card / online payments** gave cashiers a server error; the POS crashed when a payment gateway was not set up.
- **POS cart:** +/−/remove did nothing on a second variant's line; a line could inherit the previous line's variant.
- **Barcode scanning:** a scanner's Enter submitted the purchase form; late search results overwrote newer ones.
- **Three-dot menus** opened off the bottom of the page. They now open up or down, wherever they fit.
- **Edit windows** centred the amount like a table cell (Expense, Customer Group, Subscriptions and more).
- **Page heads** broke their filters onto two lines and left big gaps (Purchases, Purchase Report, Account History).

### Dark theme pass
The dark theme left light patches that made text hard or impossible to read. I fixed them across the app:
- a white strip under the page when a menu reached past the footer
- POS payment and action buttons ("Due" was white on pale pink)
- the invoice and purchase invoice pages and the Invoice Settings preview (dark around white paper; printing unchanged)
- Stock Lots product rows (white with light text)
- yellow "Returned" / "Partial" badges (light text on yellow)
- the light-mode logo showing in the sale, draft and purchase windows

---

## 🛠 How I worked and tested

- **Test like a customer first.** I used the app as the cashier, the owner and the accountant would. If a real person would be confused, slowed down or lose money, it was logged as a bug, even when nothing crashed.
- **Atomic changes.** Every money, stock or points operation runs in one database transaction; a failure rolls everything back.
- **One journal for stock and one ledger for points.** Stock moves through the central stock journal, and every points change is a row carrying the balance after it, so nothing can drift.
- **127 Playwright tests in 27 files.** They run against a throw-away copy of the database, so real data is never touched. They cover purchase, POS, FIFO, IMEI, returns and refunds, membership points, reports and profit, invoices and accounts, settings, dark mode, roles and the API, notifications and password reset.
- **Two recorded journeys (19 steps):** super admin → admin → POS → customers → a limited employee, and a FIFO journey through every way stock comes in and goes out.
- **Sweeps instead of spot checks.** Every menu page opened as admin and super admin at three screen sizes: 346 three-dot menus opened per size, 23 pages with the date filter, every edit window's fields, very long names. Nothing off-screen, cut off or wider than the screen.
- **Report figures cross-checked with SQL** against totals computed straight from the database.
- **AI as a pair programmer.** I used an AI coding assistant to trace root causes, draft fixes and write tests faster. I decided what to test, reviewed every change and checked the numbers myself.

```mermaid
flowchart LR
    P1[Purchase 1<br/>10 × cost 20<br/>sells at 30] --> L1[(Lot 1)]
    P2[Purchase 2<br/>10 × cost 15<br/>sells at 22] --> L2[(Lot 2)]
    L1 -->|oldest first| S[POS sale of 7]
    L2 --> S
    S --> R1[5 × 30 — profit 10 each]
    S --> R2[2 × 22 — profit 7 each]
```

---

## 📸 Screenshots

### Membership points

| Settings: earn and spend rates | POS: points under the customer's name |
|---|---|
| ![Membership points settings](images/membership-points-settings.jpg) | ![Points in the customer search](images/pos-points-search.jpg) |

| POS: points spent on the bill | Invoice: points discount, earned and balance |
|---|---|
| <img src="images/pos-points.jpg" width="360" alt="Points spent at the POS"> | ![Invoice with points](images/invoice-points.jpg) |

| Points history per customer |
|---|
| ![Points history](images/customer-points-history.jpg) |

### Settings and the dark theme

| Logos for light and dark, decimal places | Date filter: "All time" inside the picker |
|---|---|
| ![General settings](images/settings-logos-decimals.jpg) | ![Date filter](images/date-filter-all-time.jpg) |

| POS in the dark theme | Purchase details with the dark-mode logo |
|---|---|
| ![POS dark](images/dark-pos.jpg) | ![Purchase details dark](images/dark-purchase-details.jpg) |

| Stock lots in the dark theme | Invoice page in the dark theme |
|---|---|
| ![Stock lots dark](images/dark-stock-lots.jpg) | ![Invoice dark](images/dark-invoice.jpg) |

### Reports, returns and the POS

| Profit / Loss statement | A report per product |
|---|---|
| ![Profit / Loss](images/profit-loss.jpg) | ![Product report](images/product-report.jpg) |

| Trending Products | Sales Report |
|---|---|
| ![Trending Products](images/trending-products.jpg) | ![Sales Report](images/sales-report.jpg) |

| Stock lots (FIFO) | Purchase Report |
|---|---|
| ![Stock lots](images/stock-lots-fifo.jpg) | ![Purchase Report](images/purchase-report.jpg) |

| Invoice Settings | Role permissions |
|---|---|
| ![Invoice Settings](images/invoice-settings.jpg) | ![Role permissions](images/role-permission.jpg) |

| Purchase return by IMEI | POS: a variant's IMEIs only |
|---|---|
| ![Purchase return by IMEI](images/purchase-return-imei.jpg) | ![POS variant window](images/pos-variant-imei.jpg) |

| Sale return: starts at 0 | Sale return: refund filled in |
|---|---|
| ![Sale return](images/sale-return.jpg) | ![Refund filled in](images/sale-return-refund.jpg) |

| POS on a phone | Profit / Loss on a phone | Customer ledger |
|---|---|---|
| <img src="images/pos-phone.jpg" width="220" alt="POS on a phone"> | <img src="images/profit-loss-phone.jpg" width="220" alt="Profit / Loss on a phone"> | ![Customer ledger](images/customer-ledger.jpg) |

---

<sub>Part of my work on ReadyPOS. Screenshots use demo data. This report describes features and fixes only; no proprietary source code is included.</sub>
