# TEAS CHAT Project Notes

Last updated: July 12, 2026

## Live URLs

- Main site: `https://teaschat.vercel.app/`
- Thank-you page: `https://teaschat.vercel.app/thank-you`
- Buyers dashboard: `https://teaschat.vercel.app/buyers`
- Buyers dashboard password: `123456`

## Important Local Files

- `index.html` - main landing/chat page
- `teas-chat.html` - duplicate main landing/chat page
- `thank-you.html` - post-checkout instructions page
- `buyers.html` - password-gated buyers dashboard
- `privacy.html` - Privacy page
- `terms.html` - Terms & Conditions page
- `money-back-guarantee.html` - Money Back Guarantee page
- `vercel.json` - enables clean URLs, so `/thank-you` and `/buyers` work without `.html`

## Current Purchase Link Status

The purchase button is currently using the Stripe **test** checkout link:

```text
https://buy.stripe.com/test_6oE9BOee31vL8dG4gg
```

The live checkout link to restore later is:

```text
https://buy.stripe.com/dRmfZg6fA6xx2wy6C2dEs2o
```

The button label is:

```text
Purchase 11 Full Sets (Exact) - $100
```

## TikTok Pixel

Pixel ID:

```text
D9959HBC77U133LMMKIG
```

Installed on:

- `index.html`
- `teas-chat.html`
- `privacy.html`
- `terms.html`
- `money-back-guarantee.html`
- `thank-you.html`

Events:

- `PageView` - base pixel
- `ViewContent` - fires on main pages
- `InitiateCheckout` - fires when the purchase button is clicked
- `Purchase` - fires on `thank-you.html`

The purchase button click waits `1500ms` before redirecting to Stripe so TikTok and Google Sheets tracking can fire first.

## Google Sheets Tracking

Spreadsheet ID:

```text
1IL7gZEZGyujBzCBw0bT8rENffUsxXkofUtFr9zFVMwc
```

Worksheet/tab name:

```text
Buyers
```

Current Sheet columns:

```text
timestamp | checkout_clicked | status | email | amount | utm_source | utm_campaign | utm_id | ttclid | page_url
```

Apps Script Web App URL:

```text
https://script.google.com/macros/s/AKfycbyytBHgXBdDgslVTUNchD30r_0hULEtY1GQcIV1Zbqamr1dKYMAyCgg-KiD3-Bdn1-f/exec
```

Website checkout-click rows are temporary. They contain:

```text
timestamp | yes | blank | blank | blank | utm_source | utm_campaign | utm_id | ttclid | page_url
```

Zapier payment rows contain:

```text
timestamp | blank | complete/paid | buyer email | amount | blank | blank | blank | blank | blank
```

Apps Script `matchPaidRows()` copies campaign data from the closest checkout-click row into the Zapier payment row, marks the payment row as `matched`, then deletes the temporary checkout-click row.

## Apps Script Requirements

The deployed Apps Script must include:

- `doPost(e)` for checkout-click rows
- `doGet(e)` with `action=buyers` for the dashboard
- `matchPaidRows()` to match Zapier rows to checkout rows
- `DASHBOARD_SECRET = '123456'`
- A time-driven trigger:

```text
Function: matchPaidRows
Event source: Time-driven
Type: Minutes timer
Interval: Every 5 minutes
```

After every Apps Script code change:

1. Save the script.
2. Deploy a new version: `Deploy > Manage deployments > Edit > New version > Deploy`.
3. Keep the `matchPaidRows` trigger active.

## Zapier Setup

Existing Zapier flow:

- Stripe payment succeeds
- Gmail sends files/materials
- Google Sheets adds payment row

Google Sheets mapping should be:

```text
timestamp        -> Stripe payment time
checkout_clicked -> blank
status           -> complete or paid
email            -> Stripe customer email
amount           -> Stripe payment amount
utm_source       -> blank
utm_campaign     -> blank
utm_id           -> blank
ttclid           -> blank
page_url         -> blank
```

Apps Script fills the blank campaign fields later.

## Buyers Dashboard

Dashboard URL:

```text
https://teaschat.vercel.app/buyers
```

Password:

```text
123456
```

Dashboard behavior:

- Loads buyer rows from Apps Script using JSONP.
- Shows newest rows first.
- Shows 10 rows per page.
- Has previous/next pagination.
- Has search across email, campaign, campaign ID, ttclid, status, and URL.
- Shows summary stats:
  - total rows
  - paid rows
  - matched rows
  - total amount

## TikTok Test URLs

Main page UTM test:

```text
https://teaschat.vercel.app/?utm_source=tiktok&utm_medium=paid&utm_campaign=test_campaign&utm_id=test_campaign_id&ttclid=test_ttclid_123
```

TikTok Pixel test URL:

```text
https://teaschat.vercel.app/?tt_test_id=D9959HBC77U133LMMKIG_1783781568
```

Thank-you test URL:

```text
https://teaschat.vercel.app/thank-you?tt_test_id=D9959HBC77U133LMMKIG_1783781568
```

## Important Git Commits

- `904e8dd` - Add buyers dashboard
- `91bc39e` - Paginate buyers dashboard
- `d0e5521` - Align checkout tracking payload with sheet columns
- `7f4cde2` - Send checkout tracking to Google Sheets
- `1ffaf40` - Add TikTok conversion events
- `6f4bcfb` - Install TikTok pixel

