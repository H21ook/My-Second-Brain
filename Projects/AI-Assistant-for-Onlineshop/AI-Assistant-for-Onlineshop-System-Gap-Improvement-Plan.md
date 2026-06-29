---
title: "System Gap & Improvement Plan"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/gap-improvement-plan/system-gap-improvement-plan.md"
---

# System Gap & Improvement Plan

## 1. Зорилго

Энэ төлөвлөгөө нь одоогийн **AI assistant for online shops** системийн ерөнхий архитектур, route map, Supabase/RLS, API/webhook, service layer, deployment notes дээр үндэслэн дутагдалтай хэсгийг тодорхойлж, MVP болон production-ready түвшинд сайжруулах ажлуудыг эрэмбэлнэ.

Системийн үндсэн зорилго:

> Онлайн дэлгүүр Facebook Page-ээ холбож, бүтээгдэхүүний каталогоо оруулан, Messenger дээр AI sales assistant-аар хэрэглэгчийн асуултад автоматаар хариулах SaaS платформ бүтээх.

---

## 2. Одоогийн суурь сайн талууд

Одоогийн систем дараах чухал суурьтай байна.

- Next.js App Router дээр public site, auth, dashboard, manage admin, API routes тусгаарлагдсан.
- Supabase Auth, Postgres, Storage, RLS ашиглаж multi-tenant суурь тавигдсан.
- Store-level role guard: `requireStoreRole()`.
- Admin guard: `requireSuperadmin()`.
- Meta Messenger webhook болон Facebook OAuth route бүтэцтэй.
- QPay payment callback/check flow орсон.
- Product import pipeline эхэлсэн.
- Redis-ийг OAuth state, deduplication, chat history, catalog cache зэрэгт ашиглахаар төлөвлөсөн.
- `ai_usage_logs`, `conversations`, `messages` зэрэг AI visibility/analytics-д хэрэгтэй table-ууд байна.

Энэ нь MVP гаргах боломжтой суурь гэсэн үг. Гэхдээ production-д найдвартай ажиллуулахын тулд доорх gap-уудыг нөхөх шаардлагатай.

---

# 3. Гол дутагдал ба сайжруулах чиглэлүүд

## 3.1 AI Model Routing & Cost Control дутуу

### Асуудал

Одоогийн chatbot pipeline Gemini ашиглахаар байгаа ч model сонголт, fallback, token budget, store-level AI limit, daily/monthly usage cap тодорхой биш байна.

Хэрэв бүх асуултыг үнэтэй model руу явуулбал SaaS margin хурдан муудна.

### Эрсдэл

- Нэг дэлгүүрийн их chat бүх platform cost-ийг өсгөнө.
- Trial хэрэглэгч abuse хийж болно.
- AI cost billing-той холбогдохгүй бол ашиггүй ажиллана.

### Сайжруулах санал

Model routing:

```ts
simple_faq          -> Gemini 3.1 Flash Lite
product_search      -> Gemini 3.1 Flash Lite
recommendation      -> Gemini 3 Flash Preview
low_confidence      -> Gemini 3 Flash Preview
premium_store       -> Gemini 3.5 Flash
admin_summary       -> Gemini 3.5 Flash
```

Store-level config нэмэх:

```ts
ai_default_model
ai_fallback_model
ai_premium_model
max_daily_ai_cost_usd
max_monthly_ai_cost_usd
max_daily_messages
is_ai_enabled
handoff_enabled
```

`ai_usage_logs` дээр дараах багануудыг заавал хадгалах:

```ts
store_id
conversation_id
model
input_tokens
output_tokens
estimated_cost_usd
latency_ms
intent
confidence
fallback_used
created_at
```

### Priority

**P0 — MVP гарахаас өмнө**

---

## 3.2 Product Search / RAG layer тодорхойгүй

### Асуудал

AI хариулахдаа product catalog-ийг яаж хайж, ямар context өгөх нь тодорхой биш байна. Бүх бүтээгдэхүүнийг prompt руу хийх нь буруу.

### Эрсдэл

- Token cost өснө.
- AI буруу бүтээгдэхүүн санал болгоно.
- Бэлэн байхгүй барааг байгаа гэж хэлэх магадлалтай.
- Том catalog-той дэлгүүр дээр систем удааширна.

### Сайжруулах санал

AI pipeline-г дараах байдлаар задална:

```txt
Messenger message
  -> normalize text
  -> detect intent
  -> product search
  -> top N product context
  -> AI response
  -> validation
  -> send reply
```

Search priority:

1. Exact SKU / product name match
2. Postgres full-text search
3. Category / attribute filter
4. Optional pgvector semantic search
5. Fallback to human handoff

Prompt-д зөвхөн top 5–10 product явуулах.

Context жишээ:

```json
{
  "products": [
    {
      "name": "iPhone 15 Pro",
      "price": 4200000,
      "stock": 3,
      "colors": ["black", "blue"],
      "url": "..."
    }
  ]
}
```

### Priority

**P0 — chatbot quality-ийн хамгийн чухал хэсэг**

---

## 3.3 AI Safety, Grounding, Guardrails дутуу

### Асуудал

AI зөвхөн тухайн дэлгүүрийн бүтээгдэхүүн, хүргэлт, үнэ, үйлчилгээний хүрээнд хариулах ёстой. General ChatGPT шиг хариулбал brand risk үүснэ.

### Эрсдэл

- Буруу үнэ хэлэх.
- Бараа байгаа гэж худлаа хэлэх.
- Дэлгүүрийн бодлогоос гадуур амлалт өгөх.
- Хэрэглэгчтэй зохисгүй хэлбэрээр харилцах.

### Сайжруулах санал

System prompt-д хатуу дүрэм оруулах:

```txt
You are a sales assistant for this store only.
Only answer using provided store profile, products, policies, and conversation context.
Do not invent price, stock, discount, delivery date, warranty, or payment terms.
If information is missing, ask a short clarification or offer human handoff.
Reply in Mongolian by default.
Keep reply short and sales-focused.
```

Validation layer нэмэх:

- Price hallucination check
- Stock hallucination check
- Product not found check
- Too long answer check
- Unsupported topic check
- Human handoff trigger

### Priority

**P0**

---

## 3.4 Human Handoff Flow бүрэн тодорхой биш

### Асуудал

AI бүх асуултыг шийдэхгүй. Confidence бага, angry customer, payment issue, complaint, custom order үед хүн рүү шилжүүлэх хэрэгтэй.

### Эрсдэл

- AI буруу хариулж хэрэглэгч алдах.
- Store owner AI-д итгэхгүй болно.
- Complaint escalation алдагдана.

### Сайжруулах санал

Conversation state нэмэх:

```ts
ai_status: "active" | "handoff_requested" | "human_active" | "closed"
handoff_reason
assigned_member_id
last_human_reply_at
```

Handoff trigger:

- AI confidence < 0.7
- Product not found 2 удаа дараалан
- Customer asks for human/operator
- Payment/refund/complaint keywords
- Profanity/angry sentiment

Dashboard дээр:

- “Needs attention” inbox
- “Take over” button
- “Return to AI” button
- Suggested AI reply

### Priority

**P0**

---

## 3.5 Webhook Queue, Retry, Deduplication илүү баталгаажуулах хэрэгтэй

### Асуудал

Webhook route `after()` ашиглан HTTP response хурдан буцаагаад дараа нь боловсруулдаг бүтэцтэй. Энэ нь энгийн үед ажиллах ч production traffic дээр queue/retry strategy тодорхой байх ёстой.

### Эрсдэл

- Vercel runtime limit дотор task тасалдах.
- Meta retry давхардсан message үүсгэх.
- AI reply алдагдах.
- Redis outage үед dedup/history доголдох.

### Сайжруулах санал

Webhook processing-г queue job болгох:

```txt
/api/webhook
  -> verify signature
  -> store inbound event
  -> enqueue job
  -> return 200

worker/job
  -> dedupe
  -> process AI
  -> send message
  -> write logs
```

MVP үед Upstash Redis queue ашиглаж болно. Дараа нь:

- Upstash QStash
- Supabase queue table
- Trigger.dev
- Inngest
- BullMQ + Redis

Dedup key:

```txt
meta_event:{page_id}:{sender_id}:{mid}
```

### Priority

**P1 — MVP дараа production hardening**

---

## 3.6 Rate Limiting ба Abuse Protection дутуу

### Асуудал

Webhook, import, payment check, AI reply, Facebook connect зэрэг endpoint-ууд abuse-д өртөх боломжтой.

### Эрсдэл

- AI cost spike
- Import DoS
- Payment check spam
- OAuth state spam
- Store-level noisy tenant бусдад нөлөөлөх

### Сайжруулах санал

Rate limit scope:

```txt
by store_id
by page_id
by psid
by endpoint
by IP for public endpoints
```

Redis-based limit:

```ts
ai_messages_per_psid_per_minute = 10
ai_messages_per_store_per_day = plan_limit
import_jobs_per_store_per_day = 5
payment_check_per_payment_per_minute = 3
facebook_connect_per_user_per_hour = 10
```

Plan-тэй холбох:

```txt
Trial: 100 AI replies/day
Basic: 1,000 AI replies/month
Pro: 10,000 AI replies/month
```

### Priority

**P0/P1**

---

## 3.7 Observability, Error Tracking, Alerting дутуу

### Асуудал

Webhook, AI, QPay, Facebook OAuth, import, cron бүгд external dependency-тэй. Алдаа гарахад owner/admin хурдан мэдэх хэрэгтэй.

### Эрсдэл

- Messenger reply quietly fail.
- QPay paid боловч subscription сунгагдахгүй.
- Cron ажиллахгүй store status буруу болно.
- Import failure хэрэглэгчид ойлгомжгүй харагдана.

### Сайжруулах санал

Structured log стандарт:

```ts
{
  event: "ai_reply_failed",
  store_id,
  conversation_id,
  provider: "gemini",
  error_code,
  latency_ms,
  retry_count
}
```

Admin dashboard metrics:

- AI success rate
- AI average latency
- Webhook failure count
- Meta Send API failure count
- QPay pending too long
- Cron last run time
- Import success/failure rate

Tools:

- Sentry
- Vercel logs
- Supabase logs
- Custom `system_events` table

### Priority

**P1**

---

## 3.8 Subscription, Billing, Quota Enforcement сул байна

### Асуудал

Subscription lifecycle байгаа ч AI usage quota, billing plan, feature gating, overage handling тодорхой биш байна.

### Эрсдэл

- Trial abuse.
- Paid user болон free user ялгарахгүй.
- AI зардал төлбөрөөсөө давна.

### Сайжруулах санал

Plan config table:

```ts
plans
- id
- name
- monthly_price_mnt
- ai_messages_limit
- products_limit
- members_limit
- facebook_pages_limit
- import_jobs_limit
- premium_model_enabled
```

Runtime check:

```txt
Before AI reply:
  check subscription active/trial
  check monthly quota
  check daily safety cap
  check store writable/AI enabled
```

Owner dashboard дээр:

- Used AI replies
- Estimated AI cost
- Plan limit
- Upgrade prompt

### Priority

**P0/P1**

---

## 3.9 Import Pipeline-д validation/limit илүү тодорхой хэрэгтэй

### Асуудал

Import нь service-role write ашиглаж, uploaded spreadsheet боловсруулдаг high-risk flow.

### Эрсдэл

- Том файл memory/time limit давна.
- Malformed XLSX DoS болно.
- Duplicate SKU/data corruption.
- Product image upload failure partial state үүсгэнэ.

### Сайжруулах санал

Import limit:

```txt
max_file_size_mb = 10
max_rows_trial = 500
max_rows_paid = 5000
max_images_per_product = 8
max_cell_length = 1000
```

Validation:

- Required fields
- SKU uniqueness per store
- Price numeric
- Stock integer
- Image URL/file type validation
- Category normalization
- Dry-run preview before commit

Processing:

- Chunked upsert
- Transaction boundary per chunk
- Import job status transition guard
- Catalog cache invalidation after success

### Priority

**P0/P1**

---

## 3.10 Facebook Token & Permission Security илүү нарийвчлах хэрэгтэй

### Асуудал

Facebook page token маш sensitive. Documentation дээр token column-г `select('*')` хийхгүй байх гэж анхааруулсан. Үүнийг code-level standard болгох хэрэгтэй.

### Эрсдэл

- Page access token leak.
- Нэг tenant нөгөө tenant-ийн Facebook connection харах.
- OAuth state/session mismatch bug.

### Сайжруулах санал

Repository rule:

```txt
Never select facebook_connections.page_access_token outside service layer.
Never use select('*') on facebook_connections.
Only service-role Meta sender can read token.
```

Service functions:

```ts
getFacebookConnectionStatus(storeId)
getPageAccessTokenForSend(storeId, pageId)
rotateOrDisconnectPage(storeId)
```

Security tests:

- Store member cannot read token
- Other store cannot read connection
- OAuth state must match same user
- Disconnected page cannot send replies

### Priority

**P0**

---

## 3.11 Testing Strategy бараг харагдахгүй байна

### Асуудал

Auth, RLS, webhook, payment, import, AI pipeline бүгд test-тэй байх ёстой. Одоогийн docs дээр testing strategy тодорхой биш байна.

### Эрсдэл

- Guard алгассан route production-д гарах.
- Service-role misuse tenant data leak үүсгэх.
- Payment idempotency эвдрэх.
- Webhook signature verification эвдрэх.

### Сайжруулах санал

Test pyramid:

```txt
Unit tests:
  - intent detection
  - cost calculation
  - prompt builder
  - product search mapping
  - payment helper

Integration tests:
  - Supabase RLS policies
  - requireStoreRole
  - webhook signature
  - QPay callback idempotency
  - import job status transitions

E2E tests:
  - signup/login
  - create store
  - import product
  - connect Facebook mock
  - receive message -> AI reply
  - billing/payment flow
```

Tools:

- Vitest
- Playwright
- Supabase local test DB
- MSW/fetch mock for Meta, QPay, Gemini

### Priority

**P1**

---

## 3.12 Admin / Ops Dashboard хангалттай гүн биш

### Асуудал

Manage routes байгаа ч platform operator-д production үед хэрэгтэй operational view бүрэн тодорхой биш байна.

### Сайжруулах санал

Admin-д нэмэх:

- Store health status
- AI usage by store
- Cost by model
- Webhook failures
- QPay reconciliation queue
- Failed import jobs
- Facebook disconnected pages
- Stores near quota
- Trial ending soon
- Manual disable AI button

### Priority

**P1**

---

## 3.13 Store Owner Onboarding Flow сайжруулах хэрэгтэй

### Асуудал

SaaS бүтээгдэхүүнд owner эхний 10 минутанд value мэдрэх ёстой.

### Сайжруулах санал

Onboarding checklist:

```txt
1. Store profile үүсгэх
2. Facebook Page холбох
3. Product import хийх
4. AI settings шалгах
5. Test message илгээх
6. AI-г live болгох
```

Dashboard home дээр progress card:

```txt
Setup progress: 3/6 completed
Next: Import products
```

### Priority

**P0/P1**

---

## 3.14 Data Privacy, Retention, Delete Flow тодорхой болгох

### Асуудал

Conversations/messages/AI logs/customer PSID зэрэг sensitive data хадгалагдана.

### Эрсдэл

- Store устгахад customer data үлдэх.
- Conversation retention хэт урт.
- Privacy page бодит flow-той таарахгүй.

### Сайжруулах санал

Policy:

```txt
messages retention: 180 days by default
ai_usage_logs: analytics only, no full customer text if possible
deleted store: purge products/messages/conversations after grace period
export/delete request: admin tool
```

DB:

- `deleted_at`
- `purged_at`
- `retention_until`

### Priority

**P1/P2**

---

# 4. Recommended MVP Scope

## MVP-д заавал байх зүйл

```txt
1. Auth + create store
2. Facebook Page connect
3. Product import/manual CRUD
4. Product-aware Messenger AI reply
5. Human handoff
6. Conversation log
7. AI usage logging
8. Basic billing/subscription gate
9. Admin view for stores/errors
```

## MVP-д түр хойшлуулах зүйл

```txt
1. Multi-channel Instagram/website widget
2. Advanced analytics
3. Full vector RAG
4. Automated order creation
5. Delivery integration
6. Complex loyalty system
7. Multi-language beyond Mongolian/basic English
```

---

# 5. Implementation Roadmap

## Phase 0 — Stabilize Core Architecture

Duration: 1–2 weeks

- [ ] Document actual `services/chatbot/*` implementation.
- [ ] Document actual `lib/redis.ts` behavior.
- [ ] Document actual `lib/import/xlsx.ts` limits.
- [ ] Confirm all protected routes/actions call proper guards.
- [ ] Remove unsafe `select('*')` from sensitive tables.
- [ ] Add `.env.example` with required variables.
- [ ] Add basic error boundary and server logging convention.

---

## Phase 1 — AI Reply Quality & Cost Safety

Duration: 2–3 weeks

- [ ] Implement intent detection.
- [ ] Implement product search before AI.
- [ ] Add prompt builder with strict grounding rules.
- [ ] Add model router.
- [ ] Add token/cost estimator.
- [ ] Write `ai_usage_logs` for every reply.
- [ ] Add store-level AI enabled/disabled setting.
- [ ] Add daily/monthly quota checks.
- [ ] Add fallback to human handoff.

---

## Phase 2 — Messenger Production Reliability

Duration: 2–3 weeks

- [ ] Strengthen webhook deduplication.
- [ ] Add queue/job processing for inbound messages.
- [ ] Add retry policy for Meta Send API.
- [ ] Add failure event logs.
- [ ] Add admin webhook failure view.
- [ ] Add rate limits by store/page/psid.
- [ ] Add test mode for connected Facebook Page.

---

## Phase 3 — Product Import & Catalog Hardening

Duration: 1–2 weeks

- [ ] Add file size and row limits.
- [ ] Add dry-run validation preview.
- [ ] Add duplicate SKU report.
- [ ] Add chunked processing.
- [ ] Add catalog cache invalidation.
- [ ] Add import error export.
- [ ] Add product search index optimization.

---

## Phase 4 — Billing, Quotas, Admin Ops

Duration: 2 weeks

- [ ] Add plan table/config.
- [ ] Link plan limits to AI/product/import/member usage.
- [ ] Add owner usage dashboard.
- [ ] Add admin usage/cost dashboard.
- [ ] Add QPay reconciliation warning.
- [ ] Add cron last-run monitoring.
- [ ] Add manual store suspend/enable AI control.

---

## Phase 5 — Testing & Security Hardening

Duration: ongoing

- [ ] Add Vitest unit tests.
- [ ] Add Playwright critical path tests.
- [ ] Add RLS integration tests.
- [ ] Add webhook signature tests.
- [ ] Add QPay idempotency tests.
- [ ] Add import validation tests.
- [ ] Add prompt regression tests.
- [ ] Add CI check: typecheck, lint, test, build.

---

# 6. Suggested Database Additions

## `store_ai_settings`

```sql
create table store_ai_settings (
  store_id uuid primary key references stores(id) on delete cascade,
  is_ai_enabled boolean not null default true,
  default_model text not null default 'gemini-3.1-flash-lite',
  fallback_model text not null default 'gemini-3-flash-preview',
  premium_model text,
  max_daily_messages integer not null default 100,
  max_monthly_messages integer not null default 1000,
  max_daily_cost_usd numeric not null default 1,
  handoff_enabled boolean not null default true,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

## `system_events`

```sql
create table system_events (
  id uuid primary key default gen_random_uuid(),
  store_id uuid references stores(id) on delete set null,
  event_type text not null,
  severity text not null default 'info',
  source text not null,
  metadata jsonb not null default '{}',
  created_at timestamptz not null default now()
);
```

## `plans`

```sql
create table plans (
  id text primary key,
  name text not null,
  monthly_price_mnt integer not null,
  ai_messages_limit integer not null,
  products_limit integer not null,
  members_limit integer not null,
  facebook_pages_limit integer not null default 1,
  premium_model_enabled boolean not null default false,
  created_at timestamptz not null default now()
);
```

---

# 7. Suggested Service Layer Structure

```txt
src/services/chatbot/
  pipeline.ts
  model-router.ts
  prompt-builder.ts
  intent-detector.ts
  product-search.ts
  response-validator.ts
  handoff.ts
  usage-logger.ts
  cost.ts

src/services/limits/
  rate-limit.ts
  quota.ts

src/services/observability/
  system-events.ts
  logger.ts
```

---

# 8. AI Pipeline Target Design

```txt
Inbound Messenger Event
  ↓
Verify Signature
  ↓
Dedupe Event
  ↓
Load Store + Facebook Connection
  ↓
Check Subscription + AI Quota
  ↓
Load Conversation State
  ↓
Detect Intent
  ↓
Search Catalog
  ↓
Build Grounded Prompt
  ↓
Choose Model
  ↓
Call Gemini
  ↓
Validate Response
  ↓
Send Messenger Reply
  ↓
Save Message + Usage Log
  ↓
Update Conversation Summary / Cache
```

---

# 9. Priority Matrix

| Area | Priority | Why |
|---|---:|---|
| Product search before AI | P0 | AI quality + cost control |
| Model routing/cost cap | P0 | SaaS margin хамгаална |
| Human handoff | P0 | AI алдааг business damage болгохгүй |
| Facebook token safety | P0 | Security critical |
| AI guardrails | P0 | Hallucination багасгана |
| Rate limits | P0/P1 | Abuse/cost spike хамгаална |
| Queue/retry | P1 | Production reliability |
| Observability | P1 | Silent failure илрүүлнэ |
| Import hardening | P1 | Data corruption/DoS хамгаална |
| Billing quotas | P1 | Paid SaaS model баталгаажна |
| Testing | P1 | Regression багасгана |
| Advanced analytics | P2 | Revenue expansion |
| Multi-channel | P2 | MVP дараа |

---

# 10. Хамгийн түрүүнд хийх 10 ажил

1. Chatbot pipeline дээр product search → AI гэсэн flow-г баталгаажуулах.
2. Gemini model router нэмэх.
3. `ai_usage_logs`-д token/cost/model хадгалах.
4. Store-level AI quota нэмэх.
5. Human handoff state нэмэх.
6. Facebook token access service layer-ээр л уншдаг болгох.
7. Webhook dedup key-г баталгаажуулах.
8. Import row/file limit нэмэх.
9. Admin-д failed webhook/AI error харах view нэмэх.
10. CI дээр typecheck/lint/test/build заавал ажиллуулах.

---

# 11. Анхааруулга

Бүх хэсгүүд дээр дэвшүүлж байгаа асуудлуудыг шалгаж үзэх хэрэгтэй. Үнэхээр ийм асуудал байна уу? Эсвэл өөр асуудал байж магадгүй. Хийхээс өмнө заавал шалгаарай.

# 12. Дүгнэлт

Одоогийн системийн хамгийн том дутагдал нь framework эсвэл database биш. Суурь архитектур хангалттай сайн байна.

Гол дутагдал нь дараах 4 хэсэгт байна:

1. AI-г хямд, найдвартай, controlled ашиглах давхарга
2. Product search/RAG grounding
3. Human handoff + operational visibility
4. Production security/reliability guardrails

Эхний MVP дээр олон feature нэмэхээс илүү **Facebook connect + product import + product-aware AI reply + human handoff + usage logging**-ийг маш сайн ажиллуулах нь хамгийн зөв.
