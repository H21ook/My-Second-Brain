---
title: "Production MVP Guide"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/gap-improvement-plan/production-mvp-guide.md"
---

# Production MVP Guide

## Энэ багц өөрчлөлт юунд тустай вэ

### 1. Facebook token security

- Page access token ил задгай log-д харагдах эрсдэл багасна.
- Disconnect/connect үед token-г зөвхөн service layer ашиглана.
- Tenant хооронд token leak болох магадлал буурна.

### 2. Import hardening

- 10MB-ээс том файл шууд зогсоно.
- Хэт том workbook memory/time spike үүсгэхгүй.
- Import pipeline илүү тогтвортой ажиллана.

### 3. Critical path хамгаалалт

- Webhook duplicate event-үүд DB дээр шүүгдэнэ.
- Rate limit нь Redis-гүйгээр Postgres дээр хязгаарлагдана.
- AI usage, system events, handoff state бүгд харагдана.

## Production ажиллуулахдаа хийх ёстой зүйлс

### A. Environment variables шалгах

- `NEXT_PUBLIC_APP_URL`
- `NEXT_PUBLIC_SUPABASE_URL`
- `SUPABASE_SECRET_KEY`
- `META_APP_ID`
- `META_APP_SECRET`
- `META_VERIFY_TOKEN`
- `GEMINI_API_KEY`
- `UPSTASH_REDIS_REST_URL`
- `UPSTASH_REDIS_REST_TOKEN`

### B. Migration-уудыг ажиллуулах

- `conversation_handoff_state`
- `store_ai_settings`
- `system_events`
- `inbound_messenger_events`
- `rate_limit_events`

### C. Facebook холбоос турших

1. Owner account-аар login хийх
2. Facebook connect эхлүүлэх
3. Page сонгох
4. Callback дээр `connected=1` харах
5. Disconnect хийх

### D. Import турших

1. Жижиг XLSX upload хийх
2. Mapping хийх
3. `Run import` дарж шалгах
4. 10MB-аас их file-ыг reject болж байгаа эсэхийг шалгах

### E. AI reply турших

1. Messenger дээр test message илгээх
2. Product нэр асуух
3. Үнэ/үлдэгдэл зөв гарч ирэхийг шалгах
4. Мэдэхгүй бараанд hallucination хийхгүй байгаа эсэхийг шалгах

### F. Handoff турших

1. Unsupported attachment илгээх
2. No-match product асуух
3. Quota exceeded тест хийх
4. Conversation дээр `handoff_requested`/`human_active` state харагдахыг шалгах

### G. Monitoring шалгах

- `system_events` дээр error entry орж байгаа эсэх
- `ai_usage_logs` дээр reply бүр logдож байгаа эсэх
- `inbound_messenger_events` дээр duplicate event бууж байгаа эсэх
- `rate_limit_events` дээр cap мөрдөгдөж байгаа эсэх

## Go-live checklist

- [ ] Facebook app, webhook, permissions verified
- [ ] Supabase migrations applied
- [ ] Redis / Supabase / Gemini env vars set
- [ ] Critical path manual test passed
- [ ] Admin dashboard accessible
- [ ] Backup / rollback plan confirmed
- [ ] `system_events` харах боломжтой
- [ ] Import / connect / webhook rate limits active
