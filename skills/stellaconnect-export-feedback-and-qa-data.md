---
name: Incrementally export feedback, coaching, and QA data
description: Pull Agent Connect reporting data (feedback responses, coaching sessions, QA reviews,
  audits, calibrations) with JWT auth and sequence-cursor pagination without tripping rate limits.
api: openapi/stellaconnect-data-return-openapi-original.yml
operations:
- get /v2/data
- get /v2/coaching/sessions
- get /v2/qa
- get /v2/qa/audits
- get /v2/qa/calibrations
---

# Export feedback and QA data (Agent Connect Data Return API)

The provider spec publishes no operationIds for these endpoints; they are identified by method +
path.

1. **Authenticate with both headers.** Data Return endpoints require the `x-api-key` header AND an
   `Authorization` header carrying a JWT signed with your company's API secret (HMAC SHA256) with a
   valid `iat` claim. See `authentication/stellaconnect-authentication.yml`.
2. **Choose the endpoint**: `GET /v2/data` (Feedbacks v2), `/v2/coaching/sessions`, `/v2/qa`
   (reviews), `/v2/qa/audits`, `/v2/qa/calibrations` on `https://api.stellaconnect.net`.
3. **Page by sequence, not by offset.** Each record carries a `sequence_id`; pass the highest one
   you have processed as the `after` query parameter to fetch only newer records. First backfill can
   use time windows (`created_at_gte`/`created_at_lte`, plus `completed_at_*` on audits/calibrations
   and `responded_at_*` on feedbacks) in ISO 8601 format, e.g. `2025-12-20T00:00:00-05:00`.
4. **Respect the rate limit**: 100 requests/minute per API key on every Data Return endpoint; a
   `429` means back off (see `rate-limits/stellaconnect-rate-limits.yml`). Persist your cursor so
   retries resume instead of re-reading.
5. **Handle errors**: `403` = auth failure (key or JWT), `500` = server error; retry with backoff
   and file a support ticket if reproducible.
