---
title: "Nextjs-BFF-Proxy-Patterns"
type: knowledge
status: active
created: 2026-07-07
updated: 2026-07-07
tags:
  - knowledge
  - programming
  - nextjs
  - bff
  - architecture
---

# Next.js BFF Proxy Patterns

## Summary

Browser гадаад backend-үүд рүү шууд хандахгүй — Next.js route handler-ууд (**Backend-for-Frontend** давхарга) бүх REST дуудлагыг зуучилна. Токен/cookie server талд үлдэж, лог нэг цэгт төвлөрч, upstream URL-ууд client bundle-д орохгүй. Гол хэрэгжүүлэлт нь upstream бүрд нэг **catch-all `[...path]` proxy route** + **per-upstream URL helper функц** хос юм.

## Problem

Олон микросервис (auth, core API, payment, digital signature...) бүхий системд client шууд хандвал: токен browser-т ил гарна, CORS тохиргоо сервис бүрд хэрэгтэй болно, refresh/логийн логик client-д тархана, upstream URL-ууд `NEXT_PUBLIC_*` болж bundle-д baked болно. BFF давхарга эдгээрийг нэг server-side цэгт цуглуулна.

## Core Concepts

### 1. Catch-all `[...path]` proxy route

Upstream бүрд нэг route directory: `app/backend/<upstream>/[...path]/route.ts`. Дэмжих HTTP method бүрийг export хийж, бүгдийг нийтлэг handler руу дамжуулна:

```ts
// app/backend/payment/[...path]/route.ts
import { handle } from "@/core/http/backend-handler";
import { getPaymentUrl } from "@/core/config";

export async function POST(req: NextRequest, ctx: RouteContext) {
  return handle(req, ctx, getPaymentUrl());
}
export async function GET(req: NextRequest, ctx: RouteContext) {
  return handle(req, ctx, getPaymentUrl());
}
```

Нийтлэг `handle(req, ctx, upstreamBase)` нь client-ийн замыг upstream endpoint болгон угсарна:

```ts
const { path } = await ctx.params;              // ["orders", "123"]
const endpoint = `/${path.join("/")}${search}`; // "/orders/123?x=y"
// fetch(upstreamBase + endpoint, { headers: серверийн auth header-ууд })
```

Ингэснээр шинэ endpoint нэмэхэд BFF код өөрчлөгдөхгүй — route нь бүхэл upstream-ийг тольдоно. API version-ууд тусдаа route-аар салдаг (`public-api/v1/[...path]`, `public-api/v2/[...path]`).

### 2. Per-upstream URL helper

Upstream бүрд нэг функц: `getAuthUrl()`, `getBackendUrl()`, `getPaymentUrl()`, `getDigitalSignatureUrl()` маягийн naming. Хоёр чухал дүрэм:

- **Call-time уншина, module-level const биш** — Next.js `.env` файлуудыг module ачаалагдсанаас хойш уншиж болзошгүй тул `const X = process.env.X` хэлбэрийн top-level утга хуучирч (stale) болно; мөн test override хийх боломж нээгдэнэ.
- **Тестийн override нэг цэгт** — E2E үед бүх upstream-ийг mock server руу чиглүүлж болно.

```ts
export function getPaymentUrl(): string {
  if (process.env.E2E_TEST === "1") return "http://127.0.0.1:9999";
  return process.env.PAYMENT_URL || "";
}
```

Base URL-д API version baked байвал (`.../payment-api/v1`) client path дээр version давхар нэмэхгүй гэдгийг helper-ийн doc comment-д тэмдэглэх нь ойлголцлын алдааг багасгадаг.

### 3. WebSocket-ууд BFF-ийг тойрно

Next.js route handler нь request/response амьдралын мөчлөгтэй тул урт наслах socket холболт барихад тохирохгүй. Тиймээс:

- REST дуудлага → BFF proxy (server env, browser-т үл харагдана).
- WebSocket/socket.io → browser **шууд** socket server рүү холбогдоно, URL нь `NEXT_PUBLIC_*_SOCKET_URL` env-ээр client-д ил өгөгдөнө.

Үр дагавар: socket endpoint client-д ил тул authentication-ыг socket protocol өөрөө (токен handshake г.м.) хийх ёстой — BFF-ийн cookie давхарга энд хамгаалахгүй.

### 4. Env-var-per-upstream deploy шаардлага

Upstream нэмэх бүрд deploy орчинд шинэ env зарлагдана:

- Server-only: `AUTH_URL`, `BACKEND_URL`, `PAYMENT_URL`, `DIGITAL_SIGNATURE_URL`, ...
- Socket байвал нэмээд: `NEXT_PUBLIC_..._SOCKET_URL` (build/runtime-д client-д орно).

`.env` нь ихэвчлэн gitignored тул шинэ upstream-ийн env-г prod-д зарлахаа мартвал зөвхөн runtime-д (хоосон base URL) илэрдэг. Шинэ proxy route нэмсэн commit бүрд deploy note үлдээх нь зүйтэй.

## When to Use

- Олон upstream микросервистэй, cookie/token-д суурилсан auth-тай Next.js апп.
- Токеныг browser-т гаргахгүй байх шаардлагатай үед (httpOnly cookie + server fetch).
- Логийн/observability-ийн нэг цэг хэрэгтэй үед.

## When Not to Use

- Streaming/realtime холболт (WebSocket, SSE-ийн зарим кейс) — шууд холболт (энэ тэмдэглэлийн §3).
- Ганц public API-тай жижиг апп — нэмэлт hop нь илүүц байж болно.
- Edge-д маш латенц-мэдрэг зам — BFF hop нэмэлт RTT өгнө.

## Common Mistakes

- Upstream URL-ыг module-level const-оор унших → env хожуу ачаалагдахад хоосон/stale утга.
- Socket URL-ыг BFF-ээр дамжуулахыг оролдох → route handler дээр холболт тасарна; эхнээсээ `NEXT_PUBLIC_*` шууд холболтоор төлөвлө.
- Version-baked base URL дээр client path-д version дахин нэмэх → `/v1/v1/...` маягийн 404.
- Шинэ upstream-ийн env-г deploy орчинд зарлахгүй орхих.
- Catch-all route-д зөвхөн GET/POST export хийчихээд upstream-ийн PUT/PATCH/DELETE-г мартах — method бүр тусдаа export шаарддаг.

## Trade-offs

- **+** Токен browser-т хэзээ ч гарахгүй; refresh/лог/header угсралт нэг цэгт; CORS хэрэггүй; upstream URL нууц.
- **−** Хүсэлт бүрд нэмэлт hop (латенц); socket/streaming-д тохирохгүй; upstream бүрд env bookkeeping; BFF handler bug бүх endpoint-д нөлөөлнө.

## Examples

Бодит хэрэгжүүлэлт: [[E-Geree-v3-Networking-BFF]] — `src/core/config/index.ts` (URL helper-ууд), `src/app/backend/digital-signature/[...path]/route.ts` (proxy route + socket bypass comment), `src/core/http/backend-handler.ts` (нийтлэг `handle`).

## Related Notes

- [[Nextjs-Folder-Structure]] — `core/http` / `core/config` давхаргын байршил.
