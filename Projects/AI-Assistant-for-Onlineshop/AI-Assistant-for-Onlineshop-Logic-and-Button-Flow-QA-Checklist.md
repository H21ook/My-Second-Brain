---
title: "Logic and Button Flow QA Checklist"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/plans/Logic and Button Flow QA Checklist.md"
---

# Logic and Button Flow QA Checklist

Date: 2026-06-27
Branch: `qa-logic-buttons-checklist`
Commit: `47c820e fix: dedupe inbound webhook events`

## Goal

Төслийн логик код болон тэдгээр лүү хөтлөх UI buttons, links, forms, redirects-ийг static/build/test түвшинд шалгаж, эвдрэлтэй эсвэл эрсдэлтэй flow илэрвэл тусдаа branch дээр засах.

## Scope

- Public pages: home, privacy, not-found, error retry.
- Auth flows: login, signup, forgot password, update password, OAuth/auth callback redirects.
- Dashboard routing: store resolver, store-scoped dashboard, mobile bottom tabs, sidebar/store switcher.
- Product flows: list/search/pagination, create/edit, quick edit bottom sheet, delete, image upload/delete, CSV/import flow, column mapping/start import.
- Conversations: list/detail links, takeover/return-to-AI actions, attachment links.
- Settings: general settings, AI settings, members add/remove, Facebook connect/disconnect/page select.
- Billing: subscription banner CTA, new payment submit, pending payment check/cancel/copy/bank app links.
- Manage/admin: stores, payments review buttons, users/superadmin toggle, audit links.
- API/logic routes: Facebook connect/callback, QPay callback/check, webhook, cron subscription lifecycle, import template/run.

## Out Of Scope / Limits

- Full browser click-through E2E was not run in this pass.
- External-provider success paths still need real credentials/session/data for runtime confirmation: Supabase auth, Facebook OAuth/webhook, QPay callback/check, Gemini/API calls.
- This pass proves routes/actions are type-safe, compile, and are wired statically; it does not prove third-party production callbacks succeed with live credentials.

## Checklist

- [x] Create/use separate branch for QA/fixes: `qa-logic-buttons-checklist`.
- [x] Query code graph with Graphify for routes, buttons, forms, server actions, redirects.
- [x] Inventory `href=`, `action=`, `router.push`, `redirect()`, `NextResponse.redirect`, `onClick=` usage under `src/app` and `src/components`.
- [x] Run lint.
- [x] Run unit tests.
- [x] Run TypeScript no-emit check.
- [x] Run production build with network access for Next font fetching.
- [x] Investigate lint warning from webhook route.
- [x] Fix confirmed webhook dedupe integration issue.
- [x] Add targeted route tests for new and duplicate Messenger webhook events.
- [x] Re-run verification after fix.
- [x] Commit fix in a small logical commit.
- [x] Run `graphify update .` after code changes.

## Process And Evidence

1. Graphify query mapped the Next.js route/action/button graph and surfaced the same relevant areas covered in this checklist: dashboard routes, products/actions, billing/actions, import actions, settings/Facebook actions, conversations actions, manage/admin routes, API redirects, and webhook logic.
2. Static inventory scanned link/action/button markers across `src/app` and `src/components`, including public links, dashboard tabs/sidebar, product CTAs, settings forms, billing buttons, admin actions, redirects, and route handlers.
3. `npm run lint` initially passed with one warning: unused `eventKey` in `src/app/api/webhook/route.ts`.
4. `npm run test` initially passed: 3 test files, 6 tests.
5. First sandboxed `npm run build` failed only because `next/font` could not fetch Google fonts without network access.
6. Re-running `npm run build` with network access passed and generated all expected app/API routes.
7. `npx tsc --noEmit` passed after test typing was corrected.

## Finding

### Fixed: Messenger webhook dedupe helpers were not wired into POST flow

`collectWebhookEvents()` generated an `eventKey`, and `src/services/chatbot/inbound-events.ts` already provided `enqueueInboundMessengerEvent`, `markInboundMessengerEventProcessing`, `markInboundMessengerEventProcessed`, and `markInboundMessengerEventFailed`. The webhook route destructured `eventKey` but did not use it, so Meta webhook retries could process the same Messenger event more than once.

Impact: duplicate AI replies and duplicate message/usage persistence were possible during webhook retries or duplicate deliveries.

## Action Taken

- Updated `src/app/api/webhook/route.ts` so each Messenger event is inserted into `inbound_messenger_events` before processing.
- Duplicate insert result now skips processing and still returns a successful webhook response to Meta.
- New events are marked `processing`, then `processed`; caught failures are marked `failed` with the error message.
- Added `src/app/api/webhook/route.test.ts` covering:
  - New Messenger event is queued, processed, and marked processed.
  - Duplicate Messenger event is skipped and not sent to the chatbot pipeline.

## Verification Results

- `npx vitest run src/app/api/webhook/route.test.ts`: PASS, 1 file, 2 tests.
- `npm run test`: PASS, 4 files, 8 tests.
- `npm run lint`: PASS, no warnings.
- `npx tsc --noEmit`: PASS.
- `npm run build` with network access: PASS.
- Build route manifest included all expected public, dashboard, manage, auth, and API routes.
- `git diff --check`: PASS; only line-ending warning from Git on Windows (`LF will be replaced by CRLF`) was reported.
- `graphify update .`: PASS; graph rebuilt with 1411 nodes, 2913 edges, 75 communities.

## Result

Static/build/test-level QA passed after one real logic issue was fixed. The checked buttons, links, forms, server actions, redirects, and route handlers are compiling and type-safe, and the app production build emits the expected route tree.

## Remaining Runtime Checks Recommended

- Run authenticated browser E2E on seeded/local Supabase data for dashboard navigation and CRUD flows.
- Test Facebook connect/callback and webhook with a real Meta app/page in a staging environment.
- Test QPay payment create/check/callback with staging credentials.
- Test import run on realistic XLSX/CSV files in a real browser session.
