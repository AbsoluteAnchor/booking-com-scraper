[Booking Com Scraper](https://apify.com/skipper_lume/booking-com-scraper?fpr=data)

# Booking.com Scraper

Scrape hotel data, prices, ratings, and reviews from Booking.com. Search by destination with dates or extract detailed property information from specific hotel URLs.

## Features

- **Search mode**: Find hotels by destination + check-in/check-out dates
- **Detail mode**: Extract full property info from hotel URLs
- **Rich filters**: Star rating, price range, property type, review score, sort order
- **Complete data**: Prices, discounts, reviews, amenities, room types, photos, coordinates
- **Multi-currency**: Get prices in any currency Booking.com supports (USD, EUR, GBP, etc.)
- **Multi-language**: Results in any Booking.com language (en-us, de, fr, nl, etc.)
- **Pagination**: Automatically scrapes multiple pages of search results
- **Anti-blocking**: Powered by [Crawlee](https://crawlee.dev/) — automatic fingerprint randomization, session pool with proxy rotation, block detection & retry

## Input — Search Mode

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | string | `search` | Set to `search` |
| `destination` | string | required | City, region, or hotel name |
| `checkin` | string | +30 days | Check-in date (YYYY-MM-DD) |
| `checkout` | string | +32 days | Check-out date (YYYY-MM-DD) |
| `adults` | integer | 2 | Number of adults |
| `rooms` | integer | 1 | Number of rooms |
| `currency` | string | USD | Currency code |
| `maxPages` | integer | 5 | Max pages to scrape (~25 hotels/page) |
| `sortBy` | string | — | Sort: price, distance, review_score_and_price, etc. |
| `stars` | array | — | Filter by stars, e.g. [4, 5] |
| `minPrice` | integer | — | Min price per night |
| `maxPrice` | integer | — | Max price per night |
| `minReviewScore` | integer | — | Min score: 60/70/80/90 |

### Example Search Input

```
{
    "mode": "search",
    "destination": "Amsterdam, Netherlands",
    "checkin": "2026-06-01",
    "checkout": "2026-06-03",
    "adults": 2,
    "currency": "EUR",
    "maxPages": 3,
    "stars": [4, 5],
    "sortBy": "review_score_and_price"
}
```

## Input — Detail Mode

| Field | Type | Description |
| --- | --- | --- |
| `mode` | string | Set to `detail` |
| `urls` | array | List of Booking.com hotel URLs |

### Example Detail Input

```
{
    "mode": "detail",
    "urls": [
        "https://www.booking.com/hotel/nl/waldorf-astoria-amsterdam.html",
        "https://www.booking.com/hotel/nl/the-dylan-amsterdam.html"
    ]
}
```

## Output — Search Results

Each hotel in search results includes:

| Field | Description |
| --- | --- |
| `name` | Hotel name |
| `url` | Direct Booking.com link |
| `price` | Total price for stay |
| `price_currency` | Currency code |
| `original_price` | Price before discount |
| `discount_percent` | Discount percentage |
| `review_score` | Guest score (0-10) |
| `review_count` | Number of reviews |
| `review_word` | Rating label (Superb, Very Good, etc.) |
| `stars` | Star rating (1-5) |
| `location` | Neighborhood/area |
| `distance_from_center` | Distance from center |
| `free_cancellation` | Free cancellation available |
| `breakfast_included` | Breakfast included |
| `room_type` | Recommended room type |
| `image` | Main photo URL |

## Output — Hotel Details

Detail mode returns comprehensive property data:

| Field | Description |
| --- | --- |
| `hotel_id` | Booking.com hotel identifier (from URL) |
| `description` | Hotel description (up to 1000 chars) |
| `address` | Full street address |
| `city` | City name |
| `country` | Country code (GB, NL, ES, etc.) |
| `stars` | Official star rating (1-5, extracted from page data) |
| `review_score` | Overall guest score (0-10) |
| `review_count` | Total number of reviews |
| `review_word` | Rating label (Exceptional, Superb, Very Good, Good) |
| `review_scores` | Per-category scores: Facilities, Cleanliness, Comfort, Location, etc. |
| `amenities` | Popular hotel facilities (Free Wifi, Pool, Parking, etc.) |
| `images` | Gallery photo URLs in max resolution |
| `rooms` | Room types with bed configurations and max occupancy |
| `checkin_time` / `checkout_time` | Check-in/out times (e.g. "3:00 PM" / "11:00 AM") |
| `property_type` | Hotel, Hostel, Apartment, B&B, etc. |

## Proxy Configuration

Booking.com has strong anti-bot protection. **Residential proxies are recommended** for reliable results:

```
{
    "proxyConfiguration": {
        "useApifyProxy": true,
        "apifyProxyGroups": ["RESIDENTIAL"]
    }
}
```

## Cost Estimate

Using Apify residential proxies:

- **Search**: ~$0.10-0.15 per page (25 hotels)
- **Detail**: ~$0.05-0.10 per hotel page
- **Typical run** (5 pages, 125 hotels): ~$0.50-0.75

## How It Works

This actor uses **Crawlee PlaywrightCrawler** with:

- **Browser fingerprint randomization** — each request uses a unique browser fingerprint
- **Session pool** — automatic proxy/session rotation when blocks are detected
- **Smart retries** — blocked requests are retried with fresh sessions (up to 6 attempts)
- **404 detection** — non-existent hotel pages are reported cleanly without wasting retries

## Tips

1. **Dates matter**: Prices vary significantly by dates. Always specify check-in/check-out.
2. **Use filters**: Star rating and price filters reduce pages and speed up scraping.
3. **Residential proxies**: Datacenter proxies get blocked quickly by Booking.com.
4. **Currency**: Set currency to avoid conversion issues — Booking.com returns prices in the selected currency.
5. **Memory**: Search mode works with 2 GB. Detail mode with multiple hotels may need 4 GB.