# Dream Space — Wall Panel Ordering App

A real full-stack app: a Node.js/Express backend that stores orders in
`orders.json`, plus the customer storefront and a password-protected admin
dashboard. Every file sits in one flat folder (no subfolders) so it uploads
cleanly through GitHub's web interface, which doesn't support drag-and-drop
folders.

## What's new in this version

- **Categories** — Wall Panels, PU Stone, UV Sheet, Gypsum Board (edit/add
  categories by changing the `category` field on any product in `server.js`).
- **17 products** using your real photos.
- **Payment instructions** — checkout shows your Mobile Money number and name.
  There's no real payment gateway wired in (that needs a provider account +
  API access this project doesn't have) — customers pay manually and you
  confirm it.
- **Unique order codes** (e.g. `DS-7F3K2Q`) generated per order.
- **Order tracking page** (`track.html`) — customers enter their code to see
  live status, no login needed.
- **Ratings & reviews** — customers can leave a star rating + comment,
  shown publicly on the storefront with an average.
- **Auto-generated receipts** — the moment you mark an order "payment
  confirmed" in the dashboard, a receipt becomes available at
  `/receipt.html?code=DS-XXXXXX` (also linked from the customer's tracking
  page and your admin table). It's a clean, printable page — "Download /
  Print" uses the browser's own Print → Save as PDF.
- **Optional automatic email** — if a customer gives an email at checkout
  *and* you've set up `EMAIL_USER`/`EMAIL_PASS` (see below), their receipt
  is emailed to them automatically the instant you confirm payment. Without
  those two variables set, everything still works — the customer just
  retrieves the receipt themselves from the tracking page instead of it
  arriving by email.

## What's inside

```
dream-space-app/
  server.js          <- backend: serves pages + order/review API
  package.json
  orders.json         <- orders get written here (kept private — not servable)
  reviews.json        <- ratings/reviews get written here (kept private, served via API)
  index.html          <- customer storefront
  admin.html          <- your order dashboard
  track.html          <- customer order-tracking page
  receipt.html         <- printable/downloadable receipt page
  *.jpg                <- 17 product photos
```

Only the HTML pages and photos are ever served directly to visitors.
`server.js`, `package.json`, `orders.json`, and `reviews.json` are never
exposed as raw files, even though they live in the same folder — the server
code explicitly whitelists which files are public. Reviews are still public,
but only through the `/api/reviews` endpoint, not as a raw file.

## Run it locally

You need [Node.js](https://nodejs.org) 18+ installed.

```bash
cd dream-space-app
npm install
npm start
```

Then open:
- **http://localhost:3000** — the storefront customers use
- **http://localhost:3000/admin.html** — your dashboard (default admin key: `changeme123`)

**Change the admin key before going live:**

```bash
ADMIN_KEY=your-own-secret npm start
```

## Turning on automatic receipt emails (optional, free)

This uses your own Gmail account to send the email — no paid service needed.

1. Turn on 2-Step Verification on your Google account (Settings → Security).
2. Go to https://myaccount.google.com/apppasswords and create an "app password" (a 16-character code, different from your normal Gmail password).
3. Set two environment variables (on Render: Settings → Environment Variables):
   - `EMAIL_USER` — your Gmail address
   - `EMAIL_PASS` — the 16-character app password (not your real password)
4. Redeploy. From then on, any order where the customer gave an email gets
   a receipt sent automatically the moment you mark it "payment confirmed."

If you skip this, the site works exactly the same — the receipt is just
something the customer fetches themselves from the tracking page, rather
than landing in their inbox.

## Deploying to Render.com

1. Push this folder to a GitHub repo (drag all files at once — no folders to worry about now).
2. On Render, create a new "Web Service" from that repo.
3. Build command: `npm install` — Start command: `npm start`.
4. Add environment variable `ADMIN_KEY` with your own secret (and, optionally, `EMAIL_USER`/`EMAIL_PASS` as above).
5. Add a persistent disk (Render's "Disks" feature) mounted at `/opt/render/project/src` so `orders.json` survives restarts.

**On "the tracking code should never leave the system, no matter what":**
that guarantee depends on step 5, not on the app's code. Without a
persistent disk (i.e. on Render's free tier), the whole `orders.json` file —
every order, every code — gets wiped on each restart or redeploy, no matter
how the code is written. The $7/month plan + disk is the actual fix. If you
want something that survives even without paying for that, the next step up
is moving storage from a local JSON file to an external database (e.g. a
free Supabase Postgres project) — a bigger change, happy to build it if you
want to go that route instead.

## Honest limitations of this version

- **Storage is a JSON file**, not a database. Fine for a small business —
  migrate to Postgres/MySQL later if volume grows.
- **Admin auth is a single shared key**, not individual logins.
- **No payment processing** — orders are captured as "please confirm and
  collect payment," same as a phone/WhatsApp order today.

## Editing products or prices

Open `server.js` and edit the `PRODUCTS` array near the top.
