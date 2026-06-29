---
title: "Төслийн одоогийн төлөвийн товч"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/gap-improvement-plan/project-current-status-mn.md"
---

# Төслийн одоогийн төлөвийн товч

## Юу хийдэг вэ

Энэ төсөл нь онлайн дэлгүүрүүдэд зориулсан AI assistant платформ. Гол зорилго нь Facebook Messenger дээр ирсэн хэрэглэгчийн асуултад тухайн дэлгүүрийн бараа, үнэ, үлдэгдэл, дүрэм журмын хүрээнд автомат хариу өгөх.

## Одоогоор зөв ажиллаж байгаа гол үйлдлүүд

### 1. Дэлгүүр үүсгэх ба эрхийн хяналт

- Хэрэглэгч өөрийн дэлгүүр үүсгэж чадна.
- `requireStoreRole()`-оор owner/admin эрхийг сервер талд шалгадаг.
- `requireSuperadmin()`-оор platform operator access-ийг тусад нь хамгаалдаг.
- Deleted store дээрх ажиллагаа хаагддаг.

### 2. Facebook Page холболт

- Owner Facebook Page connect flow-ийг эхлүүлж чадна.
- OAuth state, session, callback шалгалт ажилладаг.
- Холбогдсон page-ийн token-г зөвхөн service layer ашигладаг.
- Page status dashboard дээр харагдана.

### 3. Бараа оруулах ба удирдах

- Spreadsheet import ажиллаж байна.
- SKU, үнэ, үлдэгдэл, мөрийн validation хийдэг.
- Chunked upsert ашигладаг.
- Import дууссаны дараа catalog summary cache цэвэрлэгддэг.
- Product CRUD dashboard дээр ажилладаг.

### 4. Messenger AI хариу

- Messenger webhook орж ирсэн мессежийг боловсруулдаг.
- User message-ийг Redis history-д хадгалдаг.
- Product intent илэрвэл catalog дээр эхлээд хайлт хийдэг.
- Барьцгүй үед таамаглахгүй, “олсонгүй” гэсэн аюулгүй хариу өгдөг.
- Gemini prompt-д дэлгүүрийн баримжаатай catalog grounding ордог.

### 5. Human handoff

- Conversation дээр `ai_status` хадгалдаг.
- `handoff_reason`, `assigned_member_id`, `last_human_reply_at` зэрэг state хадгалагдана.
- Unsupported attachment, quota exceed, product not found үед handoff хүсэлт үүсдэг.
- Dashboard дээр `Take over` болон `Return to AI` action бий.
- Conversation list болон detail дээр AI status харагдана.

### 6. AI quota ба store settings

- Store тус бүрийн AI тохиргоо `store_ai_settings` дээр хадгалагдана.
- `is_ai_enabled`, `handoff_enabled`, `max_daily_messages`, `max_monthly_messages` ажиллаж байна.
- Env-based cap нь fallback байдлаар үлдсэн.
- AI ашиглалтын өдөр/сарын хязгаар хэтэрвэл reply зогсдог.

### 7. Usage logging ба observability

- `ai_usage_logs` дээр token, model, latency, tool call log хадгалагдана.
- `system_events` дээр pipeline-ийн алдаа, quota hit, no-match, invalid token зэрэг event log хийгддэг.
- Admin / manage хэсэгт usage statistics харагддаг.

### 8. Inbound webhook queue ба duplicate хамгаалалт

- Webhook event-үүд `inbound_messenger_events` дээр queue хэлбэрээр ордог.
- Duplicate delivery-г Postgres unique key-ээр барьдаг.
- Processing status нь `pending`, `processing`, `processed`, `failed` гэж хадгалагдана.
- Энэ замаар Redis дээрх ачааллыг нэмээгүй.

### 9. Rate limiting

- `rate_limit_events` дээр DB-backed rate limit ажиллаж байна.
- Facebook connect, import run, payment check, webhook ingestion дээр cap тавьсан.
- Redis rate limit ашиглаагүй.

## Ямар давуу талтай болсон бэ

- AI зөвхөн дэлгүүрийн бараанд тулгуурлан хариулдаг.
- Буруу таамаг, hallucination багассан.
- Quota болон store-level settings илүү хяналттай болсон.
- Duplicate webhook delivery-үүд safe байдлаар шүүгддэг.
- Abuse хамгаалалт нэмэгдсэн.
- Silent failure-үүдийг `system_events`-оор хянах боломжтой болсон.

## Одоогоор үлдэж буй ажлууд

- Facebook token security-ийг улам хатуу болгох.
- Import pipeline hardening-ийг илүү нарийвчлах.
- Admin / Ops dashboard-ийг өргөтгөх.
- Retention / privacy delete flow хэрэгжүүлэх.
- Test coverage-ийг нэмэх.

## Товч дүгнэлт

Төсөл одоо дараах үндсэн зүйлсийг асуудалгүй хийж байна:

- дэлгүүрийн эрхийн хяналт
- Facebook Page холболт
- бараа импорт ба CRUD
- Messenger дээр product-aware AI хариу
- human handoff
- store-level quota болон AI settings
- usage / system event logging
- inbound webhook duplicate хамгаалалт
- rate limiting

Энэ нь MVP-ийн үндсэн урсгалыг дэмжих хангалттай суурь болж байна.
