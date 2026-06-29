---
title: "System Gap Improvement Worklog"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/gap-improvement-plan/system-gap-improvement-worklog.md"
---

# System Gap Improvement Worklog

## 2026-06-24

- Created branch `gap-improvement-plan` in the repo.
- Reviewed `system-gap-improvement-plan.md` in the vault.
- Confirmed the main chatbot gap is still AI grounding, quota control, and safer failover.
- Started implementing reusable catalog search helpers and AI safety scaffolding in the codebase.

## Current implementation focus

- Pre-AI product grounding from the catalog.
- Conservative daily/monthly message caps until per-store settings exist.
- Safer routing and handoff behavior when the assistant cannot answer reliably.

## Next steps

- Finish the chatbot pipeline updates.
- Verify lint/build results.
- Commit changes in goal-sized chunks.

## 2026-06-24 update

- Implemented conservative AI quota checks before generating replies.
- Added preflight catalog grounding for product-related questions.
- Added no-match fallback so the bot can fail safely instead of guessing.
- Reused the grounding summary inside the Gemini prompt to keep responses anchored.
- Local ESLint passed for the touched files.

## 2026-06-24 handoff update

- Added `conversations.ai_status`, `handoff_reason`, `assigned_member_id`, and `last_human_reply_at`.
- Added chatbot conversation-state helpers for request/human-active/return-to-AI transitions.
- Wired the Messenger pipeline to request handoff on unsupported attachment, quota exhaustion, and no-match product intents.
- Added dashboard conversation controls for `Take over` and `Return to AI`.
- Added an AI status badge in the conversations list and detail page.
- Local ESLint passed for the new and modified files.

## 2026-06-24 quota/settings update

- Added `store_ai_settings` as the per-store AI configuration table.
- Added a chatbot settings loader with sensible defaults when the row is missing.
- Switched the pipeline quota checks to store-scoped caps instead of hard-coded env caps.
- Kept handoff gating configurable through `handoff_enabled`.
- Local ESLint passed for the quota/settings files.

## 2026-06-24 observability update

- Added `system_events` as a lightweight platform event log.
- Added an observability helper for best-effort event writes.
- Wired the chatbot pipeline to log unsupported attachments, quota hits, no-match cases, invalid Facebook tokens, usage-log failures, and top-level pipeline failures.
- Local ESLint passed for the observability files.

## 2026-06-24 webhook queue update

- Added `inbound_messenger_events` as a Postgres-backed webhook queue/dedup table.
- Moved duplicate suppression away from Redis for inbound webhook events.
- Kept Redis only for chat history, not for webhook event load.
- The webhook route now enqueues events by key and processes them in `after()` with DB status tracking.
- Local ESLint passed for the webhook queue files.

## 2026-06-24 rate limiting update

- Added a Postgres-backed `rate_limit_events` table.
- Added a small rate-limit helper that does one count query and one insert on allowed requests.
- Wired rate limiting into Facebook connect, import runs, payment checks, and webhook event ingestion.
- Redis was not expanded for rate limiting; chat history remains the only Redis-backed chatbot state.
- Local ESLint passed for the rate-limit files.

## 2026-06-24 MVP hardening update

- Removed a stray OAuth credential log from the Facebook flow.
- Added a 10MB hard cap to the import runner before workbook parsing.
- This keeps oversized imports from consuming extra memory and runtime.
- Local ESLint passed for the touched files.

## 2026-06-24 bulk-image-comments plan

- Wrote a unified plan for fast product entry, Messenger image support, and Facebook comment-to-chat.
- The plan keeps Redis limited to short-lived cache/state and moves new workflow state to Postgres.
- Added a clear task order: product entry first, then image support, then comments.

## 2026-06-24 product entry update

- Added a shared bulk product parser for pasted rows.
- Added a bulk paste UI on the new product page.
- Kept quick add in place and placed it side-by-side with bulk paste.
- This makes adding many products faster than Excel upload for day-to-day catalog work.
- Redis usage was not expanded; this change is UI + validation + batch write only.
- Local ESLint passed for the new product-entry files.

## 2026-06-25 bulk preview update

- Bulk paste UI now shows a live preview before submit.
- Users can see row count, error count, and the first few parsed products immediately.
- This reduces mistakes before the DB write happens.
- Redis usage was not expanded.
- Local ESLint passed for the touched bulk-entry files.

## 2026-06-25 messenger image update

- Messenger image attachments now persist in Supabase Storage and link back to `messages` through `message_attachments`.
- The chatbot replies with a short acknowledgment when a customer sends an image.
- Conversation detail pages now render stored images inline so support staff can inspect them without leaving the dashboard.
- Redis was not expanded for image handling; storage + Postgres carry the attachment state.
- Local TypeScript typecheck passed with `tsc --noEmit`.

## 2026-06-25 facebook comment update

- Page webhook now recognizes Facebook feed comment changes in addition to Messenger messages.
- Comment events are stored in facebook_comment_events and deduped through Postgres, not Redis.
- The system sends a private reply to the commenter with a short handoff message.
- Local TypeScript typecheck passed with tsc --noEmit.



## 2026-06-25 comment simplification update

- Removed comment persistence and dashboard tracking.
- Comment webhook now sends only a private greeting reply that asks how to help.
- Redis was not expanded.
- Local TypeScript typecheck passed with 	sc --noEmit.


## 2026-06-25 chatbot reply flow note

- `webhook/route.ts` дээрээс орсон Messenger event-ийг эхлээд page token / store active эсэхээр шүүж байна.
- `conversation.ai_status` `human_active` бол AI дахиж хариу өгөхгүй, хүний takeover-ыг хүндэлж байна.
- Text message ирвэл Redis дээрх `chat history`-г авч Gemini-ээр хариу үүсгэнэ.
- Product-related асуултад эхлээд catalog search хийж, тохирох бараа олдохгүй бол no-match reply буцаадаг.
- AI quota хэтэрсэн бол quota reply илгээж handoff хүсэлт үүсгэнэ.
- Image attachment ирвэл Supabase Storage-д хадгалаад `[image]` placeholder-тайгаар dashboard messages-д бүртгэнэ.
- `GET_STARTED` postback дээр мэндчилгээний reply явуулж, conversation history-д хадгална.
- Redis-ийг зөвхөн chat history болон mid dedup-д ашиглаж, шинэ workflow state-д өргөжүүлэхгүй.
