# CampusGo Vendor App

A standalone app for vendors to upload and manage available meals, gas
items, laundry items, and stationery — separate from the main
CampusGo customer app, but talking to the same backend.

## Run it

```
npm start
```

This serves `index.html` at **http://localhost:4000** (no build step,
no dependencies to install — `npx serve` runs on the fly).

You can also just double-click `index.html` to open it directly in a
browser, but running it through `npm start` avoids browser quirks
around `file://` pages making network requests.

## Before you run it

Open `index.html` and check the two constants near the top of the
`<script>` block:

- `API_BASE` — must point to your running CampusGo backend
  (`server.js`). Defaults to `http://localhost:5000/api`.
- `CAMPUSGO_APP_URL` — where your main CampusGo customer app runs, so
  the "← Back to CampusGo" link goes somewhere real. Defaults to
  `http://localhost:3000`.

## Signing in

This app reuses your existing admin login (`POST /api/admin/login`,
checked against `ADMIN_PASSWORD` in your backend's `.env`), plus a
**business name** you type in on first use. There's still no separate
per-vendor account system — anyone with the admin password can sign
in as any business name. Tokens are stored in the server's memory, so
they're invalidated whenever the backend restarts; if that happens
mid-session, this app will detect the failed request and drop you
back to the login screen automatically.

## Item ownership

Every item you add is tagged with your business name. You can only
**edit or delete** items tagged with the exact same business name
(case-insensitive) — items from other businesses show as "Not yours"
with no edit/delete controls, and the server rejects those requests
even if someone bypasses the UI (see `registerCrudRoutes`'s
`ownerField` option in `server.js`). Items from before this feature
existed (or added directly to the database) have no owner and can't
be edited or deleted through this app at all until claimed.

Run `schema_update_vendor_ownership.sql` against your database before
using this — it adds the `vendor_name` column these checks rely on.

## Linking it from CampusGo

The main CampusGo app links out to this app by URL — not by loading
it as part of its own bundle. Point that link at wherever this app
ends up running:

- **Local dev:** `http://localhost:4000` (the default from `npm start`)
- **Deployed:** whatever URL you host this folder at (Vercel, Netlify,
  a static file host, etc. — it's just static HTML/CSS/JS, no server
  required beyond serving the file)
