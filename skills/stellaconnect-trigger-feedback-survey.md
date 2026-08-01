---
name: Trigger a customer feedback survey from any system
description: Send an Agent Connect standard feedback request after a customer service interaction,
  with correct auth, channel matching, and reporting-safe flags.
api: openapi/stellaconnect-requests-openapi-original.yml
operations:
- createRequest
---

# Trigger a customer feedback survey (Agent Connect)

Use the Requests API to fire a customer survey on behalf of the employee who handled an
interaction.

1. **Authenticate.** Send the `x-api-key` header on every call. Use the Test API Key while
   validating, then switch to the Production API Key (both from Settings > Integrations in Agent
   Connect). HTTPS only; unauthenticated requests fail with `403`.
2. **Call `createRequest`** — `POST https://api.stellaconnect.net/v1/requests` with a JSON body:
   - `channel` (required) must match a channel configured on the platform (defaults: `phone`,
     `chat`, `email`).
   - Identify the employee by `employee.email` **or** `employee.custom_id` — it must match how the
     user is set up in Agent Connect.
   - `customer.name` / `customer.email` personalize and address the survey.
   - Set `ext_interaction_id` to your system's interaction id and `external_url` to deep-link the
     ticket; both surface in the Agent Connect UI.
   - `tags` (max 20, values truncated at 2,000 chars) carry call reason/disposition;
     `custom_properties` is a single-level string map for filtering and QA selection.
   - Set `do_not_send: true` when logging an interaction without emailing a survey, so interaction
     counts stay accurate.
3. **Handle errors** per `errors/stellaconnect-problem-types.yml`: `400` = fix the body, `403` =
   check the key, `500` = retry later and file a support ticket if reproducible. There is no
   idempotency key — avoid blind retries of successful sends (dedupe on your
   `ext_interaction_id`).
4. **Service recovery follow-ups** use `POST /v1/recoveries` (no operationId in the provider
   spec), referencing the original request via `initial_feedback_request_id` or
   `ext_interaction_id`.
