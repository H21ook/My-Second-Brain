---
title: "Deployment and Ops"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/Deployment and Ops.md"
---

# Deployment and Ops

This note covers the environment, schedules, and operational assumptions visible from the repository.

## Runtime And Hosting

- Next.js app with App Router
- Supabase for auth, database, storage, and service-role operations
- Vercel-style cron invoking `/api/cron/subscription-lifecycle`
- Redis for ephemeral state, deduplication, and cached summaries

## Configuration Files

### `vercel.json`

- Cron job:
  - path: `/api/cron/subscription-lifecycle`
  - schedule: `0 1 * * *`

### `next.config.ts`

- Turbopack root is set to the repository root
- No additional app-specific runtime behavior is visible here

### `supabase/config.toml`

- Local Supabase project configuration
- Auth, storage, realtime, studio, and seed settings are defined here
- `site_url` and `additional_redirect_urls` show the local callback shape used during development

## Required Environment Variables

### Supabase

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`
- `SUPABASE_SECRET_KEY`

### App URLs

- `NEXT_PUBLIC_APP_URL`

### Meta / Facebook

- `META_APP_ID`
- `META_APP_SECRET`
- `META_VERIFY_TOKEN`

### QPay

- `QPAY_USERNAME`
- `QPAY_PASSWORD`
- `QPAY_INVOICE_CODE`
- `QPAY_BASE_URL` optional, defaults to the merchant API base URL

### Gemini / AI

- `GEMINI_API_KEY`

### Cron / operations

- `CRON_SECRET`

## Operational Notes

- Webhook and callback endpoints are designed to return quickly and push heavier work into after-response processing or downstream jobs.
- The Messenger webhook path assumes Meta may retry deliveries, so deduplication is necessary.
- QPay callback handling is intentionally idempotent and does not treat the callback as authoritative.
- Import jobs can take a long time and use service-role writes, so they are operationally sensitive.
- Subscription lifecycle work runs nightly and also purges storage objects for deleted stores.
- Redis is part of the runtime path for OAuth state, QPay token caching, chat history, and deduplication.
- The `gap-improvement-plan` branch reduced Redis pressure by moving webhook/event state to Postgres where possible.
- Messenger images are stored in Supabase Storage and referenced from conversation messages.
- Facebook comment handling is send-only: the system replies privately and does not keep a separate comment inbox.

## Security And Reliability Risks

- Missing or misconfigured env vars can break auth, webhooks, or payment flows.
- Service-role paths bypass RLS, so deployment secrets must be tightly controlled.
- Cron security depends entirely on `CRON_SECRET`.
- Meta webhook verification depends on `META_VERIFY_TOKEN` and `META_APP_SECRET`.
- QPay requests depend on a token cache in Redis; a Redis outage can affect billing flows.
- The app currently relies on multiple external systems behaving correctly at once: Supabase, Redis, Meta, QPay, and Gemini.

## Suggested Operational Checklist

- Verify env vars are present in every deployment environment.
- Confirm Meta and QPay callback URLs are set to the production host.
- Confirm Redis connectivity before enabling webhook and OAuth flows.
- Confirm cron is firing at the expected time.
- Confirm storage buckets and policies match the deployed Supabase project.
- Confirm service-role credentials are not exposed to client bundles.
- Confirm the `message-attachments` bucket exists if image messages are enabled.
- Confirm Meta page token and private reply permissions are valid if comment replies are enabled.

## Missing Pieces

- Need verification: exact deployment target and build pipeline outside the repository.
- Need verification: production secrets management and rotation process.
- Need verification: monitoring/alerting setup for cron, webhook failures, and payment reconciliation.
- Need verification: whether production uses the same Supabase project shape as local dev or a separate schema baseline.
