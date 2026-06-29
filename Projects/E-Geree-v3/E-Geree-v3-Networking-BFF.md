---
title: "06 — Сүлжээ ба BFF"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
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
- `index.ts` — upstream URL helper-ууд (`getAuthUrl`, `getBackendUrl`, `getBackendUrlV2`)

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

## Auth actions
`src/actions/auth-actions.ts` — server action-ууд.
Граф ажиглалт: `switchProfileAction()` → `setTokens()` (профайл солих нь токен шинэчилдэг).

Дэлгэрэнгүй route жагсаалт: [[E-Geree-v3-Routing]]
