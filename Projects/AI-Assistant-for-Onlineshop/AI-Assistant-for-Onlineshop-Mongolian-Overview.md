---
title: "Төслийн ерөнхий тойм"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/Overview_mn.md"
---

# Төслийн ерөнхий тойм

Энэ note нь төслийг бүрэн ойлгомжтойгоор Монгол хэлээр тайлбарлана. Зорилго нь кодын нарийн ширийнийг биш, төслийн юу хийдэг, яаж ажилладаг, ямар урсгалуудтай, одоо ямар төлөвтэй байгааг шууд ойлгох боломж өгөх юм.

## Энэ төсөл юу вэ

`ai-assistant-for-onlineshop` бол онлайн дэлгүүрүүдэд зориулсан AI sales assistant SaaS платформ.

Системийн гол үүргүүд:

- дэлгүүр үүсгэх
- Facebook Page холбох
- барааны каталог оруулах
- Messenger дээр ирсэн хэрэглэгчийн асуултад AI-аар хариулах
- барааны үнэ, үлдэгдэл, SKU, ангилал дээр тулгуурлан зөв хариу өгөх
- AI шийдэж чадахгүй үед хүний takeover руу шилжүүлэх
- төлбөр, subscription, import, webhook, monitoring зэрэг operational урсгалыг ажиллуулах

Өөрөөр хэлбэл энэ бол зөвхөн chatbot биш. Энэ нь:

- store management
- Facebook integration
- catalog management
- AI assistant
- billing / quota
- admin / ops visibility

гэсэн олон хэсгийг нэгтгэсэн систем юм.

## Техникийн стек

Энд ашиглагдаж буй гол технологиуд:

- Next.js 16 App Router
- React 19
- TypeScript 5
- Tailwind CSS 4
- Supabase Auth, Postgres, Storage
- `@upstash/redis`
- `@google/genai`
- `xlsx`
- `zod`
- `react-hook-form`
- Radix UI суурьтай shadcn-style компонентууд
- `recharts`, `lucide-react`, `sonner`, `date-fns`

## Төслийн бүтэц

### `src/app/`

Next.js route tree болон route handler-ууд.

Эндээс харагддаг гол хэсгүүд:

- public marketing pages
- auth flow
- store dashboard
- admin / manage area
- API routes

### `src/components/`

UI болон feature component-ууд.

Гол бүлгүүд:

- `ui/` - үндсэн component primitive
- `dashboard/` - dashboard widget, form
- `products/` - бараа import/edit/upload UI
- `billing/` - төлбөрийн UI
- `manage/` - admin / ops UI

### `src/lib/`

Shared server helper болон utility code.

Энд:

- auth guard
- Supabase client helper
- Redis helper
- payment/subscription helper
- audit helper
- import helper
- validation schema

### `src/services/`

Бизнес логик голчлон энд төвлөрдөг.

Чухал service-ууд:

- Meta / Facebook integration
- chatbot pipeline
- QPay integration
- rate limiting
- observability / system events

### `supabase/`

Database, migration, policy, seed, config файлууд.

Энэ хэсэг нь multi-tenant хамгаалалт, RLS, storage bucket, trigger, lifecycle logic-ийн гол эх сурвалж.

## Гол route-ууд

### Public

- `/` - marketing homepage
- `/privacy` - privacy page

### Auth

- `/auth/login`
- `/auth/signup`
- `/auth/forgot-password`
- `/auth/callback`
- `/auth/update-password`

### Dashboard

- `/dashboard` - хэрэглэгчийн хамгийн сүүлийн visible store руу шилжүүлнэ
- `/dashboard/new` - шинэ store үүсгэнэ
- `/dashboard/[storeId]` - store dashboard home
- `/dashboard/[storeId]/billing`
- `/dashboard/[storeId]/conversations`
- `/dashboard/[storeId]/conversations/[conversationId]`
- `/dashboard/[storeId]/products`
- `/dashboard/[storeId]/products/new`
- `/dashboard/[storeId]/products/[productId]/edit`
- `/dashboard/[storeId]/products/import`
- `/dashboard/[storeId]/products/import/[jobId]`
- `/dashboard/[storeId]/settings`
- `/dashboard/[storeId]/settings/members`
- `/dashboard/[storeId]/settings/ai`
- `/dashboard/[storeId]/settings/facebook`
- `/dashboard/[storeId]/settings/facebook/select-page`

### Admin / management

- `/manage`
- `/manage/stores`
- `/manage/stores/[storeId]`
- `/manage/payments`
- `/manage/audit`
- `/manage/users`

### API routes

- `/api/webhook`
- `/api/cron/subscription-lifecycle`
- `/api/facebook/connect`
- `/api/facebook/callback`
- `/api/qpay/callback`
- `/api/stores/[storeId]/import/template`
- `/api/stores/[storeId]/import/[jobId]/run`
- `/api/stores/[storeId]/payments/[paymentId]/check`

## Одоогийн гол workflow-ууд

`Workflows.md` дээр workflow бүрийг шат дарааллаар нь тусад нь бичсэн. Товчоор:

- Messenger message ирэхэд store active эсэх, AI enabled эсэх, quota-гийн төлөвийг шалгана
- Text message бол Redis дээрх chat history-г авч Gemini рүү grounded prompt үүсгэнэ
- Product intent илэрвэл catalog search хийж барааны мэдээлэлд тулгуурлан хариулна
- Image attachment бол Supabase Storage-д хадгалж conversation message дээр холбоно
- AI quota хэтэрвэл reply зогсоож handoff хүсэлт үүсгэнэ
- Facebook comment бол private reply-ээр мэндчилгээ/тусламжийн хариу явуулна

## Чухал модулиуд

### `src/lib/auth/index.ts`

- `getUser()` - одоогийн Supabase хэрэглэгчийг авна
- `requireUser()` - session байхгүй бол login руу явуулна

### `src/lib/auth/store-access.ts`

- `requireStoreRole(storeId, roles)` - store membership-ийг шалгана
- `notFound()` ашигладаг тул store existence-ийг бусдад ил гаргахгүй
- `assertStoreWritable(store)` - subscription write боломжгүй бол хаана
- `requireSuperadmin()` - platform operator эрхийг шалгана

### `src/lib/supabase/server.ts`

- request cookie дээр тулгуурласан server client үүсгэнэ

### `src/lib/supabase/admin.ts`

- service-role client үүсгэнэ
- RLS-г bypass хийдэг тул зөвхөн authorized server-side path дээр ашиглах ёстой

### `src/lib/subscriptions.ts`

- trial эхлүүлэх
- subscription сунгах
- subscription event log үүсгэх

### `src/lib/payments.ts`

- payment reference code үүсгэх
- төлбөр paid гэж тэмдэглэх
- subscription сунгах

### `src/services/chatbot/pipeline.ts`

Messenger дээрх хариулах гол урсгал энд байна.

Энд:

- sender / postback / attachment шалгана
- store active эсэхийг шалгана
- AI setting болон quota шалгана
- product intent илэрвэл catalog search хийнэ
- Gemini reply үүсгэнэ
- human handoff шаардлагатай нөхцөлд state шилжүүлнэ
- reply, usage, event log-уудыг хадгална

### `src/services/meta/oauth.ts`

- Facebook Page connect flow
- OAuth state үүсгэх / шалгах
- page list авах
- page webhook subscribe хийх

### `src/services/meta/send-api.ts`

- Messenger рүү text message илгээх
- typing indicator явуулах
- private reply илгээх
- transient error дээр нэг удаа retry хийх

## Data/Auth flow

1. Middleware session refresh хийнэ.
2. Protected route дээр `requireUser()` эсвэл `requireStoreRole()` ажиллана.
3. Dashboard route store membership-ийг шалгаад зөв store руу оруулна.
4. Store дээр хамаарах бүх write action app-level guard + DB/RLS-аар хамгаалагдана.
5. AI usage, conversations, messages, system events зэрэг лог database дээр харагдана.
6. Redis-г зөвхөн богино настай state-д ашиглаж, шинэ workflow state-д хэт тэлэхгүй байх чиглэл баримталж байна.

## `gap-improvement-plan` branch дээр хийсэн гол өөрчлөлтүүд

Энэ branch дээр дараах сайжруулалтууд хийгдсэн:

- Messenger chatbot нь барааны catalog-д тулгуурлан хариулдаг болсон
- AI quota болон store-level AI settings нэвтэрсэн
- human handoff state нэмэгдсэн
- webhook duplicate delivery-г Postgres-backed state-р илүү найдвартай болгосон
- rate limit-ийг Redis-ээс хэт хамаарахгүй байдлаар хянадаг болсон
- Messenger image attachment support нэмэгдсэн
- Facebook comment дээр private reply хийх хамгийн энгийн урсгал ажиллаж эхэлсэн
- system events болон usage logs-оор visibility сайжирсан

## Ямар давуу талтай болсон бэ

- AI хариу нь store-ийн каталогоос context авдаг болсон
- Hallucination багасах суурь тавигдсан
- AI зардлыг хянах боломж нэмэгдсэн
- Human handoff илүү тодорхой болсон
- Inbound webhook reliability сайжирсан
- Redis дээрх ачааллыг шинээр нэмэхгүй байх зарчим баттай болсон
- Image message-үүдийг dashboard дээр харах боломжтой болсон

## Одоо үлдсэн гол ажил

- Facebook token / permission hardening-ийг улам сайжруулах
- Import pipeline-ийг илүү хурдан, илүү найдвартай болгох
- Production monitoring / ops dashboard-ийг сайжруулах
- Privacy / retention / delete flow-ийг бүрэн тодорхой болгох
- Test coverage-ийг нэмэгдүүлэх

## Production-д гарахын өмнө шалгах зүйлс

- Meta app, webhook, permissions зөв эсэх
- Supabase migration-ууд production дээр бүрэн очсон эсэх
- Redis, Supabase, Gemini env var-ууд тохирсон эсэх
- Import, connect, webhook, payment flow-ууд manual test-ээр батлагдсан эсэх
- Admin dashboard ажиллаж байгаа эсэх
- Backup / rollback plan тодорхой эсэх

## Товч дүгнэлт

Энэ төсөл бол:

- Facebook connect
- product catalog
- Messenger AI sales assistant
- human handoff
- billing / quota / observability

гэсэн үндсэн ажиллагааг хамтад нь авч явдаг SaaS платформ.

`gap-improvement-plan` branch дээр хийсэн сайжруулалтаар системийн гол урсгалууд илүү тодорхой, илүү найдвартай, Redis-д бага ачаалал өгдөг бүтэцтэй болсон.

Дараа уншихад хамгийн хэрэгтэй note-ууд:

- `Workflows.md`
- `API and Webhooks.md`
- `Services Map.md`
- `Deployment and Ops.md`

