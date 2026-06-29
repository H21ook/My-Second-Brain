---
title: "Services Map"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/Services Map.md"
---

# Services Map

This note maps the non-route service layer that supports webhooks, chatbot replies, payments, and external OAuth flows.

## Core Service Areas

### Meta Integration

Files:
- `src/services/meta/oauth.ts`
- `src/services/meta/send-api.ts`
- `src/services/meta/verify-signature.ts`
- `src/services/meta/types.ts`

Responsibilities:
- OAuth connect flow for Facebook pages
- OAuth state creation/consumption using HMAC-signed payloads plus Redis nonce storage
- Page listing and webhook subscription management
- Messenger Send API message delivery
- Raw-body webhook signature verification

Main risks:
- OAuth depends on `META_APP_ID`, `META_APP_SECRET`, and `NEXT_PUBLIC_APP_URL`
- Send API errors are retried only once
- `page_access_token` must not be leaked or selected with `*`

### QPay Integration

Files:
- `src/services/qpay/client.ts`

Responsibilities:
- Create QPay invoices
- Cache and refresh QPay auth tokens in Redis
- Check invoice status as source of truth for payment confirmation

Main risks:
- Requires `QPAY_USERNAME`, `QPAY_PASSWORD`, and `QPAY_INVOICE_CODE`
- Uses a cached bearer token, so token refresh and invalidation are part of the trust model
- Callback handlers must continue to verify against QPay instead of trusting the callback alone

### Chatbot Pipeline

Files:
- `src/services/chatbot/pipeline.ts`
- `src/services/chatbot/session.service.ts`
- `src/services/chatbot/gemini.service.ts`
- `src/services/chatbot/catalog-summary.ts`
- `src/services/chatbot/catalog-tools.ts`

Responsibilities:
- Handle inbound Messenger events end-to-end
- Deduplicate webhook deliveries using Redis
- Store short chat history in Redis
- Build a compact catalog summary for prompt context
- Call Gemini and tool functions for product-aware answers
- Send replies and typing indicators back to Messenger
- Mirror conversations and AI usage into Postgres for dashboard/admin visibility
- Store Messenger image attachments in Supabase Storage and attach them to messages
- Keep Redis narrowly scoped to chat history, dedup, and short-lived cache/state

Main risks:
- Requires `GEMINI_API_KEY` for live AI replies
- Strong dependency on Redis for deduplication, chat history, and cached catalog summary
- Product catalog mutations must invalidate the catalog cache or responses go stale
- The pipeline assumes webhook processing runs after the HTTP response returns
- Image attachments add one extra storage write, but do not add Redis load

### Shared Backend Helpers

Files:
- `src/lib/redis.ts`
- `src/lib/audit.ts`
- `src/lib/payments.ts`
- `src/lib/subscriptions.ts`
- `src/lib/import/xlsx.ts`
- `src/lib/constants.ts`
- `src/lib/utils.ts`
- `src/lib/validations/*`

Responsibilities:
- Redis access and cache storage
- Audit logging
- Payment confirmation and subscription extension
- Trial / lifecycle transitions
- Import workbook parsing and template generation
- Shared constants and validation schemas

Main risks:
- Several helpers are used by service-role paths, so bugs can have platform-wide impact
- Import code and payment code both touch stateful write paths and should be treated as high risk

## Call Graph Summary

- `api/webhook` → Meta signature verification → chatbot pipeline → Gemini / Send API / Redis / Supabase
- `api/facebook/connect` → OAuth state creation → Meta OAuth redirect
- `api/facebook/callback` → state consumption → token exchange → page list stash → Facebook page subscription
- `api/qpay/callback` and `api/stores/[storeId]/payments/[paymentId]/check` → QPay client → payment confirmation helpers
- `api/stores/[storeId]/import/[jobId]/run` → XLSX parser → Supabase Storage + product upserts + audit log

## Missing Pieces

- Need verification: `src/lib/redis.ts` actual connection configuration.
- Need verification: `src/services/chatbot/catalog-tools.ts` tool definitions and safety rules.
- Need verification: `src/lib/audit.ts` implementation details and write target.
- Need verification: whether any service wrappers exist outside `src/services/*` that are also part of the runtime path.
