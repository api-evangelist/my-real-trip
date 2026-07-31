---
name: Generate a MyRealTrip marketing link
description: >-
  Find a MyRealTrip product (tour/ticket, stay, or flight) and turn its product
  URL into a trackable short MyLink (myrealt.rip) for affiliate attribution.
api: https://docs.myrealtrip.com/
base_url: https://partner-ext-api.myrealtrip.com
operations:
  - POST /v1/products/tna/search
  - POST /v1/products/accommodation/search
  - POST /v1/products/flight/fare-query-landing-url
  - POST /v1/mylink
auth: Bearer API key (Authorization: Bearer <test_sk_...|live key>)
---

# Generate a MyRealTrip marketing link

Use this to create a trackable affiliate short link for any MyRealTrip product.

## Rules
- Every request: `Authorization: Bearer YOUR_API_KEY`. POST bodies use `Content-Type: application/json`.
- `POST /v1/mylink` only accepts `targetUrl` values on MyRealTrip domains.
- Responses wrap payloads in `data`; check `result.status`/`result.code`. On 429/500/503 retry with exponential backoff (1s, 2s, 4s, max 3).

## Steps

### Tour / ticket / activity link
1. `POST /v1/products/tna/search` with `{ "keyword": "<city or product>", "page": 1, "size": 20 }`. Optionally call `POST /v1/products/tna/categories` first to get a valid `category` value for the city (values differ per city — never hardcode).
2. Take an item's `productUrl` (e.g. `https://experiences.myrealtrip.com/products/{gid}`).
3. `POST /v1/mylink` with `{ "targetUrl": "<productUrl>" }`.
4. Return `data.mylink` (the myrealt.rip short URL) and `data.mylinkId`.

### Stay link
1. `POST /v1/products/accommodation/region-autocomplete` with `{ "keyword": "<region>", "isDomestic": true|false }` to get a `regionId`.
2. `POST /v1/products/accommodation/search` with `{ "regionId": <id>, "checkIn": "yyyy-MM-dd", "checkOut": "yyyy-MM-dd", "adultCount": 2 }`.
3. Use an item's `productUrl`, then `POST /v1/mylink` as above.

### Flight link
1. `POST /v1/products/flight/fare-query-landing-url` with departure/arrival airport codes and `tripTypeCd` (OW/RT/MT). It returns `data` = a flights.myrealtrip.com landing URL.
2. Pass that URL as `targetUrl` to `POST /v1/mylink`.

## Attribution
The generated MyLink carries `mylink_id`; append `utm_content` to the target URL to match conversions back to your own system via the reporting APIs.
