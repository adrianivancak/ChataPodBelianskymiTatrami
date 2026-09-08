# Chata pod Belianskymi Tatrami — full website (with backend)

```
chata-full/
├── server/                  Node.js backend — run this
│   ├── server.js
│   ├── package.json
│   ├── hash-password.js     run this to set/change the admin password
│   ├── .env.example         copy to .env and fill in
│   └── data.json            created automatically on first run
└── public/                  the website itself, served by the backend
    ├── index.html
    ├── css/style.css
    ├── js/app.js
    ├── robots.txt / sitemap.xml
    └── images/
        └── uploads/          photos uploaded from the admin panel land here
```

## Running it the first time

You need [Node.js](https://nodejs.org) installed (any recent version).

```bash
cd server
npm install
cp .env.example .env
node hash-password.js "choose-a-password-here"
```

That last command prints a line like `ADMIN_PASSWORD_HASH=$2a$10$...`.
Paste it into `.env`, replacing the empty `ADMIN_PASSWORD_HASH=` line.

Then start the server:

```bash
npm start
```

Open **http://localhost:3000** in your browser — not the `index.html` file
directly. The backend serves the site, so it needs to actually be running.

## Changing the admin password (any time later)

```bash
cd server
node hash-password.js "your-new-password"
```

Copy the printed hash into `.env` as `ADMIN_PASSWORD_HASH`, restart the
server (`npm start`). The real password is never written to disk or sent
to the browser — only this one-way hash is stored.

## Setting up real email delivery (Gmail, prefilled)

`.env.example` is already set up for Gmail — you only need to fill in
your address and an app password:

1. Turn on 2-Step Verification on the Gmail account: myaccount.google.com/security
2. Create an app password: myaccount.google.com/apppasswords
3. In `server/.env`, set `SMTP_USER` to your Gmail address and `SMTP_PASS`
   to the 16-character app password (no spaces), and update `MAIL_FROM`
   / `MAIL_TO` to match.
4. Restart the server.

Using a different provider instead (Resend, Brevo, etc.)? Just change
`SMTP_HOST` and `SMTP_PORT` to match theirs.

Until real credentials are filled in, everything still works — reservation
requests and contact messages just get printed to the server's
console/log instead of actually emailed, so nothing is lost while you're
still setting things up.

## How reviews work

Reviews are verified by **reservation reference code**, not by emailing a
secret link — this means the server never has to store or hold onto a
guest's email address at any point.

1. A guest books through the calendar. The confirmation email you receive
   immediately includes their reference code (e.g. `REZ-4528`) **and** a
   reminder link to the review page.
2. You confirm the booking in admin once you're happy to.
3. Whenever you like after their stay, you send that same reference code
   back to the guest yourself (a quick reply, a checkout-day text
   message, however you prefer).
4. The guest goes to the "Recenzie" page, types in their reference code,
   name, a 1–5 star rating, and a comment. The server checks that a
   *confirmed* booking exists with that code and hasn't already left a
   review — if so, it's accepted.
5. New guest reviews sit as pending in admin → "Recenzie" until you
   approve them; nothing goes live without you clicking "Schváliť".

**Adding reviews from other platforms:** the same admin tab has a form to
add a review manually (name, rating, comment, and an optional "source"
label like "Tripadvisor" or "Google") — these publish immediately since
you're the one entering them, no separate approval step needed.

## Cottage capacity

The site is set up for a maximum of 4 guests. The booking form only offers
combinations that add up to 4 or fewer, with a house rule that all-adult
groups cap at 3 (so "4 adults" never appears, but "2 adults + 2 children"
and similar mixed combinations do). The maximum guest count and the
capacity note under the price are both editable in admin → "Obsah" —
the dropdown regenerates its combinations automatically if you change
the number.

## Minimum / maximum stay length

Admin → "Obsah" sets the site-wide default (e.g. minimum 3 nights, no
maximum). Admin → "Dĺžka pobytu" lets you override that for a specific
date range — for example, a 7-night minimum every summer. Whichever rule
covers a guest's arrival date is the one that applies; the calendar shows
a live hint once someone picks an arrival day, and the server rejects a
request that violates the rule even if the request bypassed the visible
hint somehow.

**Last-minute deals automatically lift this for their own dates.** If
your usual minimum stay is 5 nights but a booking pattern leaves a
3-night gap, creating a last-minute deal for exactly that gap (admin →
"Last minute") also sets the minimum stay to 1 night for that specific
window — so the gap actually becomes bookable instead of being silently
blocked by your normal minimum. This only affects the deal's own date
range; your regular minimum stays in force everywhere else. Deleting the
deal removes this override along with it.

## Editing the "About" section and hero facts

Admin → "Fakty" manages the strip of facts shown right under the hero —
each one is a fully free-form value/label pair (e.g. "4" / "hostia,
kapacita chaty", "896 m" / "nad morom"). Add as many as you like, edit
either field inline, or remove one — nothing here is fixed to a specific
meaning, so it can show anything you want. The small badge above it
("★ 4.9 (12 recenzií)") is separate and still calculates itself
automatically from approved reviews.

Admin → "Obsah" holds the address used on the Kontakt page and its map,
plus the main text content. Admin → "Vybavenie" manages the amenities
list shown in "O chate" (fireplace, terrace, etc.) — add or remove items
freely, no code changes needed.

## Logo and "why this cottage" icons

Both are blank by default and show a neutral placeholder graphic until
you upload something — there's no built-in artwork to replace.

- **Logo:** admin → "Obsah" has a small logo manager at the top — upload
  an image or remove it to go back to the placeholder. It appears in the
  site's top-left navigation.
- **Highlight card icons:** admin → "Dôvody" manages the four cards in
  "Prečo práve táto chata" — edit each card's title and text inline, and
  upload an icon image per card (or leave it as the placeholder). You can
  also add new cards or remove existing ones.

## Special offers (multiple, each with its own color)

Admin → "Špeciálne ponuky" lets you create any number of promotional
banners — a title, a message, and an optional button you can turn off
entirely (useful for a pure announcement with no call to action). Each
one gets its own color from a small preset palette, and they stack on
the homepage right under the hero. Toggle any one on/off without losing
what you wrote, or delete it outright.

## Voucher / discount codes

Admin → "Zľavové kódy" creates a code with either a percentage or a fixed
€ amount off, an optional expiry date, and an optional maximum number of
uses. Guests type the code into the booking form before submitting; it's
checked and applied to the total in real time, and checked again
server-side when the reservation is actually created (so it can't be
tampered with client-side). Once a code hits its use limit or expiry, it
stops working automatically — no need to remember to disable it.

## "Okolie" (nearby) — photos and flexible details per place

Each place in Okolie can now have an optional photo (paste a URL or
upload one directly, same as the gallery) and any number of label/value
detail tags — "Dĺžka: 8 km", "Náročnosť: stredná", "Prevýšenie: 450 m",
or whatever fits that particular place (a hike, a bike route, a ski
slope). If a place has any details, visitors see a "Zobraziť viac" link
that expands them; a place with no details just shows normally.

**Already in place:** real per-page URLs (`/kalendar`, `/galeria`, etc.)
instead of JavaScript-only navigation, a `<title>` and meta description
per page, Open Graph tags, a canonical URL, basic
[schema.org `LodgingBusiness`](https://schema.org/LodgingBusiness)
structured data, `robots.txt`, and `sitemap.xml`.

**Update before going live:** `robots.txt`, `sitemap.xml`, and the
`<link rel="canonical">` / Open Graph tags in `index.html` all currently
use a placeholder domain (`chatapodbelianskymitatrami.sk`) — replace
every occurrence with your real domain once you've chosen one.

**What actually moves the needle for a small local business, roughly in
order of impact:**
1. **Google Business Profile** (business.google.com) — free, and this is
   what shows up in Google Maps and the local pack when someone searches
   "chata Ždiar". Bigger impact than most on-page tweaks.
2. **Genuine reviews** — you now have the feature for this.
3. **Backlinks** — a listing on Booking.com/Airbnb, local tourism
   association pages, or Ždiar's community site all help.
4. **Submit the sitemap** to Google Search Console once live.
5. On-page technical details (already built above) matter, but are a
   smaller lever than the above for one small property.

## Uploading photos

Admin panel → Galéria → drag photos in, or click to choose files. They're
resized in the browser, then uploaded as real files to
`public/images/uploads/` on the server.

## Using the admin panel on your phone

The admin panel is responsive — on a narrow screen it takes over the
full screen for easier tapping, and every form field stacks into a single
column. All the same tabs and actions are there, just laid out for touch.

## Hosting this for real

This needs an actual server process running continuously, so a plain
static host (Netlify, GitHub Pages) won't work on its own. Reasonable
options, roughly easiest to most capable:

- **Render.com** or **Railway.app** — simple "point at a Node project, we
  run it" free/cheap tiers, both supporting a small persistent disk
  (needed so `data.json` and uploaded photos survive restarts).
- **A small VPS** (Hetzner, DigitalOcean) for full control — install
  Node, use `pm2` to keep the process running, and a reverse proxy
  (Caddy or Nginx) for free automatic HTTPS.

Whichever you choose:

1. Set real values in `.env` on the server (never commit `.env` to a
   public repository) — including `SITE_URL`.
2. In `server.js`, uncomment `secure: true` in the session cookie config
   once the site is served over HTTPS.
3. Point your domain's DNS at the host.
4. Update the placeholder domain in `robots.txt`, `sitemap.xml`, and the
   meta tags in `index.html`.
5. Test the full guest journey — booking, confirming, review, newsletter,
   contact — from a phone on a different network before sharing the link.
