---
title: "06 — Сүлжээ (Action → Service → HTTP)"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v2
source_path: "D:/own/obsidian-vaults/E-Geree-v2-docs/06-Networking.md"
---

# Networking — 3 давхаргат request flow

[[E-Geree-v2-Home]] · өмнөх: [[E-Geree-v2-State-Management]] · дараах: [[E-Geree-v2-PDF-Editor]]

Клиент гадаад backend руу **шууд хандахгүй**. Бүх backend дуудлага 3 давхаргаар дамжина.

## 1. httpRequest (`src/lib/config/http-request/index.js`)
Доод түвшний fetch wrapper (граф дээр 41 edge-тэй god node).
- `E_GEREE_TOKEN` cookie-оос auth токен уншина
- `X-Octagon-Auth`, `Accept-Language`, `X-Forwarded-For` header нэмнэ
- `{ result: boolean, data }` буцаана

HTTP verb helper-ууд эргэн тойронд нягт бүлэг (cohesion 0.30):
`getRequest`, `postRequest`, `putRequest`, `patchRequest`, `deleteRequest`,
`postFormDataRequest`, `fetchResponse`, `logRequestError`.

## 2. Service модулиуд (`src/lib/services/<domain>.js`)
Domain бүрийн service нь `httpRequest`-ийг ороож backend endpoint-уудыг тодорхойлно
(жишээ: notification, payment, template, contract, employee services).

## 3. Server Actions (`src/lib/actions/<domain>.js`)
`"use server"` action-ууд service-ийг дууддаг. Клиент компонент эдгээрийг React Query
(`useQuery` / `useMutation`)-ээр хэрэглэнэ. [[E-Geree-v2-State-Management]]

## Auth / cookie
- Auth cookie нь **олон хэсэгт** хуваагдсан (хэмжээний хязгаараас болж).
  Үргэлж `src/utils/cookie-utils.js` helper ашиглана — шууд унших/бичих хориотой.
- Auth context: `src/components/context/auth-context.jsx`
- Auth actions: `src/lib/actions/auth.js`
- Middleware: `src/middleware.js` — JWT шалгаж `(protected)` route хамгаална.

## Backend URL-ууд (env)
`AUTH_URL`, `BACKEND_URL`, `BACKEND_URL_V2`, `MANAGE_URL`, `SERVICE_URL`,
`DIGITAL_SIGNATURE_URL`, `PAYMENT_URL`, `KYC_URL`.

Route бүтэц: [[E-Geree-v2-Routing]]
