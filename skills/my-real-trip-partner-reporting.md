---
name: Pull MyRealTrip partner revenue and reservations
description: >-
  Retrieve a marketing partner's revenue and reservation data over a date range
  for reconciliation against your own attribution system.
api: https://docs.myrealtrip.com/
base_url: https://partner-ext-api.myrealtrip.com
operations:
  - GET /v1/revenues
  - GET /v1/revenues/flight
  - GET /v1/reservations
  - GET /v1/reservations/flight
auth: Bearer API key (Authorization: Bearer <key>)
---

# Pull MyRealTrip partner revenue and reservations

Use this to reconcile MyRealTrip affiliate performance into your own dashboards.

## Rules
- Auth: `Authorization: Bearer YOUR_API_KEY`.
- Dates are `yyyy-MM-dd`; `startDate` must not be after `endDate`.
- Revenue data settles daily ~06:00 KST (prior-day). Reservation data is real-time.
- Refund/partial-refund rows carry a negative `commission` and are flagged by `closingType`.
- Match rows to your system via `linkId` (marketing link) and `utmContent`.

## Steps

### Revenue
1. `GET /v1/revenues?dateSearchType=SETTLEMENT|PAYMENT&startDate=...&endDate=...` for tour/ticket/stay commissions.
2. `GET /v1/revenues/flight?...` for flight commissions (flight-only, no reservation detail).
3. Use `commissionBase` (actual settled amount, from 2026-04-01 bookings; `null` before → fall back to `salePrice`), `commission` (VAT included), and `commissionRate`.

### Reservations
1. `GET /v1/reservations?dateSearchType=RESERVATION_DATE|TRIP_END_DATE|CANCELED_BETWEEN_DATE&startDate=...&endDate=...&page=1&pageSize=20`. Max range 6 months. Optional `statuses` filter (CONFIRM, CANCEL, FINISH, ...).
2. `GET /v1/reservations/flight?startDate=...&endDate=...` for flight bookings (PNR in `flightReservationNo`). Max range 1 month.
3. Page via `meta.pagination { page, pageSize, totalCount }`.

## Error handling
Retry 429/500/503 with exponential backoff; do not retry 400/401/403/404 — fix the request or key instead.
