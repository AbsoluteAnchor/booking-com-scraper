[Booking Com Scraper](https://apify.com/knotless_cadence/booking-com-scraper?fpr=data)

# Booking.com Scraper — Hotel Listings as JSON/CSV

Scrape Booking.com search-results pages for hotels in any destination. Returns 16 fields per listing — name, URL, price, currency, review score, stars, review count, location, distance, image, plus the search context (dates, adults, rooms, destination, scraped timestamp).

**No API key, no Booking.com login, no Selenium/Playwright.** Uses Cheerio + Crawlee on the public search-results HTML.

---

## What this actor actually does (verified against `src/main.js`)

For every destination in your input, the actor:

1. Builds a Booking.com search URL with your check-in/out dates, adults, rooms, currency, and sort order.
2. Walks the paginated results (offsets `0`, `25`, `50`…) until either `maxResults` per destination is reached or no more cards appear.
3. Parses each property card with Cheerio selectors (3-fallback chain: `[data-testid="property-card"]` → legacy `.sr_property_block` → `[data-testid="property-card-container"]`).
4. Applies your `minPrice` / `maxPrice` / `minRating` filters in-memory before pushing.
5. Writes one record per hotel to the dataset.

That's it. ~30–90 sec per destination depending on `maxResults`.

---

## Input

```
{
  "destinations": ["Paris", "Tokyo", "London"],
  "checkIn": "2026-06-15",
  "checkOut": "2026-06-18",
  "adults": 2,
  "rooms": 1,
  "maxResults": 50,
  "currency": "USD",
  "sortBy": "review_score",
  "minPrice": 80,
  "maxPrice": 400,
  "minRating": 8.0
}
```

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `destinations` | string[] | required | One Booking.com search per destination. |
| `checkIn` | `YYYY-MM-DD` | tomorrow | If empty, the actor uses tomorrow's date. |
| `checkOut` | `YYYY-MM-DD` | day after tomorrow | If empty, day after `checkIn`. |
| `adults` | int 1–10 | 2 |  |
| `rooms` | int 1–10 | 1 |  |
| `maxResults` | int 1–500 | 50 | Cap **per destination**, not total. |
| `currency` | enum | `USD` | One of: USD, EUR, GBP, JPY, AUD, CAD, CHF, CNY, RUB, INR, BRL. |
| `sortBy` | enum | `popularity` | One of: `popularity`, `price_asc`, `rating`, `review_score`, `distance`. |
| `minPrice` | int / null | null | Per-night currency-units. Filter applied client-side. |
| `maxPrice` | int / null | null | Same as above. |
| `minRating` | number 0–10 / null | null | Filter on Booking review score. |

---

## Output (one record per hotel)

```
{
  "name": "Hotel Le Bristol Paris",
  "url": "https://www.booking.com/hotel/fr/lebristol.html",
  "price": 1240,
  "currency": "USD",
  "rating": 9.4,
  "reviewCount": 1853,
  "stars": 5,
  "location": "8th arr., Paris",
  "distance": "1.2 km from center",
  "image": "https://cf.bstatic.com/.../bristol.jpg",
  "destination": "Paris",
  "checkIn": "2026-06-15",
  "checkOut": "2026-06-18",
  "adults": 2,
  "rooms": 1,
  "scrapedAt": "2026-04-29T13:30:00.000Z"
}
```

**16 fields per hotel.** All fields may be `null` when the property card doesn't expose them — Booking ships A/B card layouts and missing data is normal.

### Field notes (be honest before you buy)

- `price` is **whatever Booking displays on the card** — it can be per-night or total-stay depending on Booking's rendering for your locale and date range. Cross-check against the URL before treating as canonical.
- `stars` is counted by DOM-element heuristic (span/svg count inside the rating element). Reliable for 1–5 hotels but can return `null` or off-by-one when Booking changes the markup.
- `distance` is a free-text string (`"1.2 km from center"`, `"500 m from Eiffel Tower"`). No parsing into numeric km.
- `reviewCount` parsing strips non-digits — works in most locales but can fail on RU/JP non-Latin formats.

---

## What this actor does NOT do (be honest)

- ❌ **No detail-page enrichment.** You get the search-card data only — no room types, no amenity list, no policies, no full review text.
- ❌ **No JS rendering.** Pure Cheerio. If Booking moves a price into a JS-rendered modal (they sometimes do), that field becomes `null` until selectors are updated.
- ❌ **No proxy rotation built in.** Booking doesn't aggressively block low-volume scraping, but at scale you should add Apify's residential proxy via the platform proxy config (and budget accordingly).
- ❌ **No CAPTCHA solving.** If Booking serves a challenge, the run will return whatever it parsed before the block.
- ❌ **No deduplication across destinations.** A hotel that appears under "Paris" and under a Paris-area neighborhood lookup will appear twice.
- ❌ **No price history.** One snapshot per run. If you need price-tracking, schedule the run on Apify Cron and append to your own dataset/DB.
- ❌ **No alternative sites.** Booking.com only — not Hotels.com, Expedia, Agoda, Trivago.

---

## Common use cases

- **Travel-aggregator side projects** — daily snapshots of top 50 hotels in 10 cities for a price-tracker dashboard.
- **Market research** — competitor pricing in a destination for a hotel-chain or OTA team.
- **Affiliate content** — populate "Top 10 hotels in Tokyo under $200" listicles with fresh data.
- **Travel-agent automations** — feed search results into Slack/email when a target hotel drops below a price threshold.

---

## When this actor stops being enough

If you need any of the following, that's a custom build (we deliver):

- Detail-page enrichment (amenities, room types, policies, review text)
- Multi-site comparison (Booking + Hotels.com + Expedia + Agoda in one record)
- Daily price-history with diff alerts
- Geo-bounded search (lat/lng + radius)
- Booking.com vacation-rentals or transport-search

**Proof of delivery + urgency**: This actor has **22 lifetime production runs** as of May 2026 — part of an active 31-published portfolio (78 total). Trustpilot scraper alone: **951 lifetime runs**. Author shipped a paid 3-article series in March 2026 ($150, proxy industry). **Pilot pricing locked through May 2026** (table below).

**Sample request?** Reply `sample` to [spinov001@gmail.com](mailto:spinov001@gmail.com) and you'll receive 2 published case-study articles within 24 hours, no obligation. Fastest way to evaluate writing style + technical depth before commissioning a custom build.

| Tier | What you get | Price |
| --- | --- | --- |
| **Pilot** | 1 actor extension (e.g. detail-page enrichment), basic config, 7-day support | **$97** |
| **Standard** | Custom multi-site or scheduled snapshot pipeline + Slack/email alerts, 30-day support | **$297** |
| **Premium** | Full price-tracking dashboard + 90-day support + 1 modification round | **$797** |

**Track record:** 31 published actors on Apify (78 total in portfolio), including:

- `trustpilot-review-scraper` — 951 runs in 30d, used by data teams for review monitoring
- `reddit-discussion-scraper` — Reddit JSON-API scraper at scale
- `email-extractor-pro` — bulk website email harvester (no Hunter cap)

**Contact / sample work:**

- Email: [spinov001@gmail.com](mailto:spinov001@gmail.com)
- Blog & case studies: [https://blog.spinov.online](https://blog.spinov.online)
- All actors: [https://apify.com/knotless_cadence](https://apify.com/knotless_cadence)
- Telegram (scraping/AI tips): [https://t.me/scraping_ai](https://t.me/scraping_ai)

---

## License

MIT — fork it, modify it, ship it.