---
title: "API and Webhooks"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/API and Webhooks.md"
---

# API and Webhooks

This note summarizes the API layer by inspecting only `src/app/api/**` and `src/lib/**`.

> [!note]
> Unknown or uninspected behavior is marked as **Need verification**.

## API Route List

| Endpoint | Purpose | Input / Output Summary | External Services Used | Security / Auth | Notes |
|---|---|---|---|---|---|
| `GET /api/webhook` | Meta webhook verification handshake | Input: `hub.mode`, `hub.verify_token`, `hub.challenge` query params. Output: `200` with challenge text or `403 Forbidden`. | Meta Messenger | Shared-secret verification via `META_VERIFY_TOKEN` | No user session involved. |
| `POST /api/webhook` | Inbound Messenger event intake | Input: raw JSON webhook body + `x-hub-signature-256`. Output: JSON `{ success: true }` or error text/status. | Meta Messenger, chatbot pipeline, Gemini/SEND path via downstream services | Signature verification only | Uses `after()` so processing happens after the immediate 200 response. |
| `GET /api/qpay/callback` | QPay payment notification hint | Input: `payment_request_id` query param. Output: plain text `IGNORED`, `NOT_PAID`, `SUCCESS`, or `ERROR`, always with HTTP 200. | QPay, Supabase service role | No user auth; webhook-like callback | Treats the callback as advisory and re-checks QPay directly. |
| `POST /api/qpay/callback` | Same as GET | Same as above | QPay, Supabase service role | No user auth | Mirrors GET for compatibility. |
| `GET /api/facebook/connect` | Start Facebook OAuth connect for a store | Input: `storeId` query param. Output: redirect to Meta OAuth, or back to dashboard settings on failure. | Meta OAuth, Upstash-backed OAuth state store via `services/meta/oauth` | `requireStoreRole(storeId, ["owner"])` | Owner-only. |
| `GET /api/facebook/callback` | Finish Facebook OAuth connect | Input: `state`, `code`, optional `error`. Output: redirects to Facebook settings or page selection, with `?error=` on failure. | Meta OAuth, Supabase auth/session, Upstash-backed OAuth state store | State + session match check | CSRF gate is the OAuth state. |
| `GET /api/cron/subscription-lifecycle` | Scheduled subscription lifecycle and tombstone purge | Input: `Authorization: Bearer <CRON_SECRET>`. Output: JSON summary or error JSON. | Supabase service role, storage API | Bearer secret | Handles deletions and storage cleanup. |
| `GET /api/stores/[storeId]/import/template` | Download Excel import template | Input: `storeId` path param. Output: XLSX file download. | Excel workbook generator in `lib/import/xlsx` | `requireStoreRole(storeId)` | Read-only endpoint. |
| `POST /api/stores/[storeId]/import/[jobId]/run` | Run an import job | Input: `storeId`, `jobId`. Output: JSON success/error response with counts. | Supabase service role, storage, audit log, catalog invalidation | `requireStoreRole(storeId)` + writable store check | Long-running import flow. |
| `POST /api/stores/[storeId]/payments/[paymentId]/check` | Poll payment status for QPay | Input: `storeId`, `paymentId`. Output: JSON status (`pending`, `paid`, error). | QPay, Supabase service role, subscription helper | `requireStoreRole(storeId, ["owner"])` | Used as a safety net when callbacks are missed. |

## Purpose By Endpoint

### `GET /api/webhook`

- Verifies the Meta webhook subscription handshake.
- Returns the challenge only when the verify token matches.

### `POST /api/webhook`

- Accepts Messenger event payloads.
- Verifies the raw-body signature before parsing JSON.
- Collects batched entries and processes them asynchronously with `after()`.
- Also recognizes Facebook comment feed changes and sends a private reply that asks how to help.

### `GET` / `POST /api/qpay/callback`

- Receives QPay payment notifications.
- Looks up the referenced payment request with the service-role client.
- Re-checks the invoice status using QPay before marking anything paid.

### `GET /api/facebook/connect`

- Begins store Facebook page connection.
- Creates OAuth state and redirects to the Meta authorize URL.

### `GET /api/facebook/callback`

- Consumes OAuth state.
- Confirms the same signed-in user started the flow.
- Exchanges the OAuth code for a long-lived token.
- Fetches managed pages and stores them for the next selection step.

### `GET /api/cron/subscription-lifecycle`

- Runs the scheduled subscription lifecycle transitions.
- Purges data for stores that have reached deleted status.
- Removes storage objects from `product-images` and `import-files`.

### `GET /api/stores/[storeId]/import/template`

- Returns a prebuilt `.xlsx` template for product imports.

### `POST /api/stores/[storeId]/import/[jobId]/run`

- Claims the import job.
- Downloads the uploaded workbook from Supabase Storage.
- Parses, validates, and upserts products.
- Records row errors and job status.
- Invalidates catalog summaries and writes audit entries.

### `POST /api/stores/[storeId]/payments/[paymentId]/check`

- Manually checks a QPay invoice.
- If paid, marks the request paid and extends the subscription.
- Returns `pending` when the invoice is not yet confirmed.

## Input / Output Summary

### Common patterns

- Webhook endpoints prefer plain text or small JSON responses and often return `200` even when the upstream system should retry less aggressively.
- Admin or cron flows return JSON summaries on success and JSON error payloads on failure.
- File download endpoints return binary responses with explicit content headers.
- Store-scoped endpoints use route params plus the store auth guard.

### Endpoint-level notes

- `POST /api/webhook`: raw body is required for signature verification before JSON parsing.
- Messenger text flow remains the primary AI path, while comments use a short private reply and do not create dashboard records.
- `GET /api/qpay/callback`: reads only a `payment_request_id` query param.
- `GET /api/facebook/connect`: reads only `storeId` and redirects.
- `GET /api/facebook/callback`: reads `state`, `code`, and `error` from the query string.
- `GET /api/cron/subscription-lifecycle`: reads the authorization header only.
- `POST /api/stores/[storeId]/import/[jobId]/run`: expects a previously mapped import job and downloads the workbook from storage.
- `POST /api/stores/[storeId]/payments/[paymentId]/check`: expects a QPay-backed payment request.

## External Services Used

- Meta Messenger webhook and OAuth.
- QPay payment verification.
- Supabase Auth.
- Supabase Postgres via server and admin clients.
- Supabase Storage.
- Upstash-backed OAuth state handling in the Meta flow.
- A chatbot processing pipeline that likely calls Gemini and message-send APIs downstream of the webhook handler.
- Messenger image attachments are persisted in Supabase Storage and rendered from conversation messages.

## Shared Library Responsibilities

### `src/lib/supabase/server.ts`

- Creates a cookie-aware server Supabase client.
- Uses `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`.

### `src/lib/supabase/client.ts`

- Creates the browser Supabase client for client-side auth actions.

### `src/lib/supabase/admin.ts`

- Creates a service-role client using `SUPABASE_SECRET_KEY`.
- Disables session persistence and auto-refresh.
- Bypasses RLS, so every caller must authorize first.

### `src/lib/auth/store-access.ts`

- Enforces user, store membership, owner-only, and superadmin checks.
- Uses `notFound()` to avoid leaking store existence.
- Exposes `assertStoreWritable()` for subscription-gated write paths.

### `src/lib/payments.ts`

- Creates payment reference codes.
- Idempotently marks a payment request as paid and extends the subscription.
- Writes audit entries after successful payment processing.

### `src/lib/subscriptions.ts`

- Starts trials and extends subscriptions.
- Records subscription event rows.

### `src/lib/audit.ts`

- Need verification: full implementation not inspected here, but it is used for payment/import audit logging.

### `src/lib/import/xlsx.ts`

- Need verification: workbook parsing, template generation, and row extraction details were not read directly in this pass.

### `src/lib/redis.ts`

- Need verification: not read directly, but appears to back some stateful flows such as OAuth state or conversation storage.

## Error Handling Pattern

- Webhooks and callbacks often catch errors, log server-side details, and return a generic response to the upstream caller.
- `POST /api/webhook` returns `400` for invalid JSON and `403` for bad signatures.
- `GET /api/qpay/callback` and `POST /api/qpay/callback` never fail loudly to QPay; they return a 200 with a status string because retries are managed by the upstream system.
- `GET /api/facebook/callback` redirects back to settings with an `error` query param rather than exposing internals.
- `GET /api/cron/subscription-lifecycle` returns `401` for missing/incorrect bearer auth and `500` for RPC or purge failures.
- The import run endpoint returns `409` for state mismatches and `500` only when processing truly fails.
- The payment check endpoint returns structured JSON and downgrades failed checks to `pending` rather than throwing away the request.

## Security Concerns

- `createAdminClient()` bypasses RLS entirely. This is the main trust boundary in the codebase.
- `POST /api/webhook` depends on exact raw-body signature verification; any proxy or parser change would break security.
- Comment replies depend on the page token and private reply permission remaining valid.
- `GET /api/facebook/callback` depends on OAuth state and the current Supabase session matching the initiating user.
- `GET /api/cron/subscription-lifecycle` is only as safe as `CRON_SECRET`.
- The QPay callback is unauthenticated by design, so it must continue to validate against QPay before mutating state.
- `GET /api/stores/[storeId]/import/template` and the import run endpoint rely on app-level authorization plus service-role operations; the database layer is not the only guard.
- `POST /api/stores/[storeId]/import/[jobId]/run` processes uploaded files, so malformed spreadsheets are a practical denial-of-service risk if row limits and parsing limits are weakened.
- The Facebook connect flow stores OAuth state server-side; state retention and cleanup should remain bounded.

## TODO List

- Need verification: `src/lib/audit.ts` implementation.
- Need verification: `src/lib/import/xlsx.ts` implementation.
- Need verification: `src/lib/redis.ts` implementation.
- Need verification: the `services/meta/*` and `services/qpay/*` service wrappers, because they define the exact upstream contract.
- Need verification: the actual middleware entrypoint that wires `updateSession()` into Next.js.
- Need verification: whether import and payment endpoints have rate limiting or replay protection beyond idempotent writes.
- Need verification: whether webhook event deduplication is implemented in the chatbot pipeline or storage layer.
