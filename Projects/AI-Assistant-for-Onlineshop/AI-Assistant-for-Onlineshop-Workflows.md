---
title: "Workflows"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/Workflows.md"
---

# Workflows

Энэ note нь төслийн хамгийн чухал operational workflow-уудыг нэг дор цэгцэлнэ. Зорилго нь code-г дахин уншихгүйгээр ямар урсгал ямар дарааллаар ажилладгийг хурдан ойлгох.

## 1. Messenger чат хариулах workflow

1. `POST /api/webhook` raw body-г signature-тай нь шалгана.
2. `page` event-үүдийг авч `processMessagingEvent()` руу дамжуулна.
3. Store active, AI enabled, subscription live эсэхийг шалгана.
4. `conversation.ai_status` `human_active` бол AI зогсоно.
5. `GET_STARTED` postback бол мэндчилгээ явуулна.
6. Text message бол Redis history-г авч Gemini рүү grounded prompt үүсгэнэ.
7. Product intent илэрвэл catalog search хийнэ.
8. Product олдохгүй бол safe no-match reply буцаана.
9. Quota хэтэрвэл quota reply + handoff хүсэлт үүсгэнэ.
10. Reply илгээсний дараа `messages` болон `ai_usage_logs`-д хадгална.

## 2. Product-aware reply workflow

1. Message text normalize хийнэ.
2. Intent detector бүтээгдэхүүнтэй холбоотой эсэхийг шүүнэ.
3. Catalog search эхлээд exact SKU/name, дараа нь broader search хийнэ.
4. Олдсон product context-ийг Gemini prompt-д оруулна.
5. AI хариуг validation-аар шалгана.
6. Hallucination, stock, price эргэлзээ гарвал fallback reply ашиглана.

## 3. Human handoff workflow

1. AI no-match, quota exceeded, unsupported attachment зэрэг үед handoff reason үүснэ.
2. `conversations.ai_status` `handoff_requested` эсвэл `human_active` болно.
3. Dashboard дээр `Take over` / `Return to AI` action ашиглана.
4. Хүн takeover хийвэл AI дахиж хариу өгөхгүй.
5. Хүн буцааж AI-д шилжүүлэхэд pipeline дахин идэвхжинэ.

## 4. Image attachment workflow

1. Messenger image attachment ирнэ.
2. Image-ийг Supabase Storage-д хадгална.
3. `messages` дээр `[image]` placeholder үүсгэнэ.
4. `message_attachments` дээр file metadata хадгална.
5. Conversation detail page inline image-ээр харуулна.

## 5. Comment reply workflow

1. Facebook page comment webhook орж ирнэ.
2. System comment-ийг public reply биш private reply-р хүлээж авна.
3. Comment бичсэн хүнд “Сайн байна уу. Танд юугаар туслах вэ?” гэсэн мэндчилгээ явуулна.
4. Comment state-г DB-д бүртгэхгүй.
5. Dashboard дээр comment tracking хийхгүй.

## 6. Import workflow

1. Store owner import file upload хийнэ.
2. Template / preview / validation хийнэ.
3. 10MB hard cap-аас том file reject хийнэ.
4. Processing job нь service-role write ашиглана.
5. Success бол catalog cache refresh хийнэ.
6. Failure бол job error-ууд display хийнэ.

## 7. Rate limit and dedup workflow

1. Webhook / connect / import / payment flow бүр дээр rate limit шалгана.
2. Duplicate Messenger delivery mid-based dedup-ээр шүүгдэнэ.
3. Inbound event processing нь Postgres-backed queue/state ашиглана.
4. Redis-ийг зөвхөн chat history, OAuth nonce, cache зэрэг богино настай зүйлд ашиглана.

## 8. Production readiness checklist

1. Meta webhook signature, OAuth callback, page token тохиргоо бүрэн эсэх.
2. Supabase migration-ууд production schema дээр очсон эсэх.
3. Redis зөвхөн шаардлагатай cache/state-д ашиглагдаж байгаа эсэх.
4. `system_events`, `ai_usage_logs`, `rate_limit_events` ажиллаж байгаа эсэх.
5. Human handoff болон import flow-ууд manual test-ээр батлагдсан эсэх.

