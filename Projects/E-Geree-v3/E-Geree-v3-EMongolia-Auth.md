---
title: "E-Mongolia (DAN) Auth — архитектур ба mobile хязгаарлалт"
type: project
status: draft
created: 2026-07-07
updated: 2026-07-07
tags:
  - project
  - project/e-geree-v3
---

# E-Mongolia (DAN) Auth — архитектур ба mobile хязгаарлалт

[[E-Geree-v3-Home]] · холбоотой: [[Universal-App-Links]]

e-Mongolia (DAN) нь гуравдагч системийн нэвтрэлт (third-party auth). Энэ reference нь
одоогийн requestId/redirectUrl + tab-д суурилсан урсгал, түүний mobile browser дээрх
хязгаарлалтыг тогтмол үнэн болгож баримтжуулна.

## v3 кодын өнөөгийн байдал (баталсан)

- **v3 login нь зөвхөн email + 2FA** — `src/components/custom/login/login-form.tsx:34-58`
  (`/auth/login-by-email` BFF route). e-Mongolia-гаар нэвтрэх урсгал v3 frontend-д
  **хараахан хэрэгжээгүй**.
- **Профайл дээрх DAN холболт түр идэвхгүй** —
  `src/features/profile/components/personal-info/OwnInfoSection.tsx:54`
  ("B2-д: DAN холбох socket урсгал — түр идэвхгүй"). `user.danVerified` талбараар
  баталгаажсан эсэхийг харуулна (мөр 41).
- **Ижил төрлийн socket-wait хэв маяг GSIGN гарын үсэгт ажиллаж байгаа** —
  `src/features/documents/components/detail/DigitalSignModal.tsx:87-124`:
  create хүсэлт → room id → browser-аас socket.io-оор шууд
  (`NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL`, BFF-ээр дамжихгүй) `join-room` хийж
  `sign` event хүлээнэ. Дэлгэрэнгүй: [[E-Geree-v3-Networking-BFF]].
- **DAN нь гэрээний оролцогчийн баталгаажуулалтын төрөл** мөн —
  `src/types/enums.ts:46` (`VerificationType.DAN`); DAN талбарууд contract-create
  wizard-д ашиглагдана (`src/features/contract-create/lib/fields.ts:352-371`).

> [!note] Таамаг тэмдэглэл
> Доорх requestId/redirectUrl tab-урсгал нь e-Mongolia auth-ийн **backend/v2 талын
> одоогийн архитектур** — v3 frontend кодод байхгүй тул кодоос мөрөөр батлах боломжгүй.
> Эх сурвалж: багийн дүн шинжилгээ (Inbox тэмдэглэл, 2026-07-07-нд distill хийсэн).

## Одоогийн архитектур — requestId/redirectUrl + шинэ tab

```text
Frontend
    │ POST /start
    ▼
Backend ──► requestId + redirectUrl
    │
    ▼
Frontend (Tab A)
    ├── requestId хадгална
    ├── Socket сонсоно
    └── redirectUrl-г шинэ Tab (Tab B) дээр нээнэ
                 │
                 ▼
           e-Mongolia (auth хийнэ)
                 │
                 ▼
           Backend Callback ──► Socket Event ──► Frontend (Tab A)
```

Синхрончлолын цорын ганц механизм нь **Tab A дээрх WebSocket** — амжилт/алдааны
мэдээлэл зөвхөн socket event-ээр буцдаг.

## Яагаад mobile дээр эвдэрдэг вэ (browser хязгаарлалтууд)

Desktop browser дээр энэ урсгал ажилладаг. Mobile browser дээр дараах хязгаарлалтууд
тогтмол үнэн:

- Background tab **pause** болдог (Tab B нээгдэхэд Tab A идэвхгүй болно).
- Идэвхгүй tab-ийн **WebSocket disconnect** болох боломжтой.
- Timer болон event-үүд идэвхгүй tab дээр ажиллахгүй байж болно (throttle).
- Зарим browser (ялангуяа in-app browser) **шинэ tab нээхгүй**.
- Зарим browser санах ой чөлөөлөхөд **өмнөх tab-ийг унтраадаг** — Tab A-гийн state алдагдана.

**Дүгнэлт (тогтмол үнэн):** зөвхөн socket-д найдсан, хоёр tab-ын хооронд state
дамжуулдаг auth урсгал mobile дээр найдваргүй. Найдвартай байхын тулд синхрончлол
нь server-side хадгалсан request status дээр суурилах ёстой; socket нь зөвхөн UX
сайжруулах нэмэлт байж болно.

## Нээлттэй асуулт — status-д суурилсан урсгал (санал, батлагдаагүй)

> [!question] Шийдэгдээгүй. Доорх нь mobile асуудлыг шийдэх **саналын түвшний**
> архитектур — хэрэгжүүлэх шийдвэр гараагүй, API contract батлагдаагүй.

Гол санаа: request-ийн төлөвийг backend дээр (Redis + DB) хадгалж, frontend нь
**same-tab redirect + status шалгалт**-аар синхрончлогдоно.

```text
FE ──POST /start──► BE (Redis: PENDING, TTL≈10min · DB: auth_requests audit)
FE ◄── requestId + redirectUrl (sessionStorage-д requestId)
FE ──same-tab redirect──► e-Mongolia ──callback──► BE (Redis/DB: SUCCESS|FAILED)
BE ──302──► FE /auth/callback?requestId=...
FE ──GET /status?requestId──► SUCCESS → dashboard · FAILED → алдаа · PENDING → poll (~2s)
```

Саналын API contract:

```text
POST /api/auth/emongolia/start     → { requestId, redirectUrl }
POST /api/auth/emongolia/callback  ← e-Mongolia (signature/token шалгаад status шинэчилнэ)
GET  /api/auth/emongolia/status?requestId=... → { status: SUCCESS|FAILED|PENDING, user?, token? }
```

Хариуцлагын хуваарь (саналд):

| Component | Responsibility |
|---|---|
| Frontend | start дуудах, requestId хадгалах, status шалгах |
| Backend | request үүсгэх, callback боловсруулах |
| Redis | түр auth state (хурдан status шалгалт, TTL) |
| Database | audit + persistence (`auth_requests`) |
| Socket | зөвхөн optional UX notification — үндсэн механизм биш |
| Status API | үндсэн синхрончлолын механизм |

Нээлттэй дэд асуултууд:

- Шинэ tab хэвээр үлдээх тохиолдолд Tab A дээр `window.focus` /
  `document.visibilitychange` event-ээр status дахин шалгах хангалттай найдвартай юу?
- Mobile app / in-app browser-аас буцаж ирэх redirect-ийг [[Universal-App-Links]]
  (universal/app links)-ээр шийдэх үү?
- GSIGN гарын үсгийн socket урсгал (`DigitalSignModal.tsx`) мөн адил status-д
  суурилсан fallback шаардлагатай юу?
