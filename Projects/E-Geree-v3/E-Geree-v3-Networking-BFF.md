---
title: "06 — Сүлжээ ба BFF"
type: project
status: active
created: 2026-06-30
updated: 2026-07-07
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/06-Networking-BFF.md"
---

# Networking — BFF давхарга

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-State-Management]] · дараах: [[E-Geree-v3-PDF-Viewer]]

Клиент гадаад backend руу **шууд хандахгүй**. `src/app/backend/*` route handler-ууд
(Backend-for-Frontend) бүх дуудлагыг зуучилж токен, cookie, лог төвлөрүүлнэ.

## Fetcher-ууд (`src/core/http/`)
| Файл | Үүрэг |
|---|---|
| `core-fetcher.ts` | Үндсэн fetch wrapper (метод, body, header) |
| `server-fetcher.ts` | Server-side fetch (auth токентой) |
| `backend-handler.ts` | BFF route-уудын нийтлэг handler |
| `headers.ts` | BFF header угсралт |
| `api-client.ts` | Axios клиент |

## Тохиргоо (`src/core/config/`)
- `index.ts` — upstream URL helper-ууд (`getAuthUrl`, `getAuthUrlV2`, `getBackendUrl`, `getBackendUrlV2`, `getPaymentUrl`, `getPaymentUrlV2`, `getDigitalSignatureUrl`)
  - `getDigitalSignatureUrl` (2d-3) → `backend/digital-signature/[...path]` прокси; base `DIGITAL_SIGNATURE_URL` нь `/digital-signature-api/v1`-ээр төгссөн. GSIGN socket нь BFF-ээр ОРОХГҮЙ, browser-аас `NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL` руу шууд холбогдоно.
  - `getPaymentUrl` (Phase E, 2026-07-04) → `backend/payment/[...path]` прокси; base `PAYMENT_URL` нь `.../payment-api/v1`.
  - `getPaymentUrlV2` (2d-4) → `backend/payment-v2/[...path]` прокси; `PAYMENT_URL_V2` тодорхойлогдсон бол түүнийг, үгүй бол `PAYMENT_URL`-ийн `/v1`-ийг `/v2`-оор сольж гаргана (`src/core/config/index.ts:42-45`).

## BFF proxy route-ууд (`src/app/backend/`, 2026-07-07 байдлаар)
| Catch-all route | Upstream helper | Method-ууд |
|---|---|---|
| `auth/[...path]` | `getAuthUrl` | POST·GET·PUT·PATCH·DELETE — бүгд бодит proxy (anti-phishing тохиргоонд PUT/PATCH/DELETE хэрэгтэй болсон, 2026-07-04) |
| `auth-v2/[...path]` | `getAuthUrlV2` | POST·GET бодит; PUT/PATCH/DELETE = `okNoop` (Phase B1, 2026-07-03) |
| `public-api/v1/[...path]` | `getBackendUrl` | POST·GET·PUT·PATCH·DELETE — бүгд бодит proxy (Phase G1: `company-integration/update` PUT, `company/remove` DELETE; label CRUD-ийн PUT/DELETE 2026-07-07-нд live-баталгаажсан) |
| `public-api/v2/[...path]` | `getBackendUrlV2` | POST·GET бодит; PUT/PATCH/DELETE = `okNoop` хэвээр |
| `payment/[...path]` | `getPaymentUrl` | POST·GET·PUT·PATCH·DELETE — бүгд бодит proxy (Phase E: payment-history) |
| `payment-v2/[...path]` | `getPaymentUrlV2` | POST·GET — зөвхөн `payment/initiate` v2-т байдаг (2d-4; client талын дуудлага `src/features/documents/api/client.ts:103`) |
| `digital-signature/[...path]` | `getDigitalSignatureUrl` | POST·GET — GSIGN (2d-3): `pdf-sign-request/create` POST + `g-sign/contract/sign` GET |

Статик route-ууд (Next.js статик segment нь catch-all-оос түрүүлж таардаг): `auth/check-auth`, `auth/login-by-email`, `auth/logout`, `auth/refresh-token`, `auth/user/upload-user-image` (multipart — generic proxy `req.json()` дууддаг тул body алдана, иймд тусдаа FormData route), `file/upload`, `health`, ба session-rotation хоёр route (доор).

### Session-rotation static route-ууд (Phase B2, 2026-07-03)
`auth/user/verify-email-confirmation` ба `auth/user/merge-account` — амжилтад upstream шинэ access token (заримдаа профайл id, refreshToken) буцаадаг тул `auth/user/_shared/rotate-session.ts`-ийн `handleSessionRotatingPost` (мөр 29) httpOnly cookie-г server талд эргүүлнэ; generic proxy httpOnly cookie-д хүрч ЧАДАХГҮЙ. Token-ыг browser руу задлахгүй — амжилтад `{isOk: true, data: null}` буцаана. v2 parity: refreshToken ирээгүй бол зөвхөн access token + профайл cookie солигдож, refresh token хуучнаараа үлдэнэ.

## Auth дамжуулалт (`src/core/auth/`)
- `tokens.ts` — токен удирдлага
- `cookie-options.ts` — cookie тохиргоо (`buildAuthCookieOptions`, refresh max-age)

## Observability ба services
- `src/core/observability/logger.ts` — лог
- `src/lib/bff-observability.ts` — BFF лог/observability (`logBff`, `runAuthRouteWithObservability`)
- `src/lib/services/` — `public-api.ts`, `user-service.ts`

## Хувилбартай public API
`backend/public-api/v1/[...path]` ба `v2/[...path]` — гадаад API-г хувилбараар проксидно
(`getBackendUrl`, `getBackendUrlV2`, `getAuthUrlV2`).
v1 нь PUT/PATCH/DELETE-г бодитоор проксидно (өмнө `okNoop` байсан); v2 дээр `okNoop` хэвээр — v2 endpoint-ууд одоогоор зөвхөн POST/GET хэрэглэдэг.

## Auth actions
`src/actions/auth-actions.ts` — server action-ууд.
Граф ажиглалт: `switchProfileAction()` → `setTokens()` (профайл солих нь токен шинэчилдэг).

## Deploy шаардлага (env, 2026-07-07 байдлаар)
`.env` gitignored тул prod deploy бүрд upstream env-үүдийг гаднаас inject хийх ёстой:
- `AUTH_URL`, `AUTH_URL_V2`, `BACKEND_URL`, `BACKEND_URL_V2` — auth/core proxy-ууд.
- `PAYMENT_URL` — payment proxy-ийн base (Phase E, 2026-07-04); байхгүй бол төлбөрийн урсгал унана.
- `PAYMENT_URL_V2` — `payment-v2` proxy-ийн override; байхгүй бол `PAYMENT_URL`-ийн `/v1` → `/v2` fallback-аар автоматаар гарна (`src/core/config/index.ts:42-45`), тиймээс prod дээр v2 host нь v1-ээс өөр бол заавал inject хийнэ.
- `DIGITAL_SIGNATURE_URL` — digital-signature proxy-ийн base (2d-3, 2026-07-07); байхгүй бол тоон гарын үсгийн урсгал бүхэлдээ унана.
- `NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL` — GSIGN socket-д browser шууд холбогддог (`src/features/documents/components/detail/DigitalSignModal.tsx:60`); `NEXT_PUBLIC_*` тул **build үед** baked болно — build орчинд заавал байх ёстой.

Ерөнхий BFF proxy загварын мэдлэг: [[Nextjs-BFF-Proxy-Patterns]]

Дэлгэрэнгүй route жагсаалт: [[E-Geree-v3-Routing]]
