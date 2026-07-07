---
title: "03 — Routing (App Router)"
type: project
status: active
created: 2026-06-30
updated: 2026-07-07
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/03-Routing.md"
---

# Routing — Next.js App Router

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-Architecture]] · дараах: [[E-Geree-v3-Contract-Create-Feature]]

Бүх route `src/app/[locale]/` дотор — олон хэлийг segment-ээр зохицуулна.
Route group-ууд `()` хаалтаар layout-г ялгана.

## Хуудасны бүтэц
(2026-07-07 байдлаар disk-тэй тулгасан, branch `dev-khishigee`)
```
[locale]/
├─ (auth)/            нэвтрэх бүлэг
│  ├─ login
│  └─ register        ← хоосон directory, page.tsx алга (route resolve хийгдэхгүй)
├─ (protected)/       нэвтэрсэн хэрэглэгч (server дээр шалгана)
│  ├─ (with-sidebar)/
│  │  ├─ documents/received
│  │  ├─ documents/sent
│  │  ├─ documents/template              ← placeholder stub (div л буцаадаг)
│  │  ├─ documents/detail/[...slug]      ← гэрээний дэлгэрэнгүй
│  │  └─ profile/                        ← layout.tsx = ProfileNav tab nav
│  │     ├─ page                         ← redirect → /profile/user/personal-information
│  │     ├─ user/{personal-information, security, login-history,
│  │     │        payment-history, subscription, referral, organization}
│  │     └─ organization/{information, user-manage, teams, teams/[slug]}
│  └─ (without-sidebar)/
│     └─ contract/create/{participant,content,fields,submit}  ← wizard
└─ (public)/          нээлттэй
   ├─ (home)
   └─ blog
```

Тэмдэглэл (2026-07-07):
- `documents/detail/[...slug]` — slug = `[contractId, actionId, accessCode?]` (`src/app/[locale]/(protected)/(with-sidebar)/documents/detail/[...slug]/page.tsx:3`). Дэлгэрэнгүй: [[E-Geree-v3-Contract-Detail]].
- `/profile` index нь `/profile/user/personal-information` руу redirect хийдэг (v2 parity, `profile/page.tsx`); profile бүх дэд хуудас `profile/layout.tsx`-ийн ProfileNav tab nav дотор амьдардаг.
- И-мэйл солих/нэгтгэх нь тусдаа route БИШ — `personal-information` хуудасны dialog-ууд (2026-07-03 Phase B2).
- Хуучин `pdf-test` route устсан (2026-07-07 байдлаар disk дээр алга).
- Sidebar-ын label мод (`src/components/layout/nav-labels.tsx`) `(with-sidebar)` layout-д байрладаг, `documents/received`-ийн `labelId` searchParam-тай холбогддог — [[E-Geree-v3-Label-Sidebar]].

## BFF route handlers (src/app/backend/)
- `auth/[...path]` — auth proxy (catch-all, `getAuthUrl()`). PUT/PATCH/DELETE-г бодитоор дамжуулдаг (anti-phishing тохиргоонд шаардлагатай; `backend/auth/[...path]/route.ts:14-25`, 2026-07-04 Phase C-д okNoop-оос сольсон).
- `auth/{check-auth, login-by-email, logout, refresh-token}` — static auth route-ууд.
- `auth/user/{verify-email-confirmation, merge-account}` — session-rotation static route-ууд: амжилтад шинэ access token + профайл cookie тавьдаг (`backend/auth/user/_shared/rotate-session.ts`). Generic catch-all proxy httpOnly cookie-г шинэчилж чадахгүй тул static байх ёстой; token browser руу задардаггүй (2026-07-03 Phase B2).
- `auth/user/upload-user-image` — multipart static override: generic proxy `req.json()` дууддаг тул FormData алдагддаг; static segment нь catch-all-оос түрүүлж таардаг (`backend/auth/user/upload-user-image/route.ts:7-12`).
- `auth-v2/[...path]` — `getAuthUrlV2()` upstream (2026-07-03 Phase B1); GET/POST л дамжина, PUT/PATCH/DELETE = `okNoop`.
- `digital-signature/[...path]` — GSIGN тоон гарын үсгийн микросервис (`getDigitalSignatureUrl()`), POST+GET л (`backend/digital-signature/[...path]/route.ts`). Socket нь BFF-ээр дамждаггүй — browser-аас шууд холбогддог (2026-07-07, Contract Detail 2d-3; [[E-Geree-v3-Contract-Detail]]).
- `payment/[...path]` — төлбөрийн upstream (`getPaymentUrl()`, 2026-07-04 Phase E); PUT/PATCH/DELETE-г мөн бодитоор дамжуулдаг (`backend/payment/[...path]/route.ts:14-25`).
- `payment-v2/[...path]` — төлбөрийн `/v2` микросервис (`getPaymentUrlV2()`), GET/POST л; одоогоор зөвхөн `payment/initiate` энд байдаг (2d-4).
- `file/upload` — файл байршуулах
- `public-api/v1/[...path]` — үндсэн backend proxy (`getBackendUrl()`); PUT/PATCH/DELETE-г бодитоор дамжуулдаг (`backend/public-api/v1/[...path]/route.ts:16-26`; label CRUD, company-integration хэрэглэдэг — PUT нь upstream-д ажилладаг нь 2026-07-07-нд live баталгаажсан, [[E-Geree-v3-Label-Sidebar]]).
- `public-api/v2/[...path]` — v2 backend proxy (`getBackendUrlV2()`); GET/POST л, PUT/PATCH/DELETE = `okNoop`.
- `health` — эрүүл мэндийн шалгалт

Дэлгэрэнгүй: [[E-Geree-v3-Networking-BFF]]
