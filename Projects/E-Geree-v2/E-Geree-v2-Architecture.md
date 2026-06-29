---
title: "02 — Архитектур"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v2
source_path: "D:/own/obsidian-vaults/E-Geree-v2-docs/02-Architecture.md"
---

# Архитектур

[[E-Geree-v2-Home]] · өмнөх: [[E-Geree-v2-Overview]] · дараах: [[E-Geree-v2-Routing]]

## Давхаргат бүтэц
```
Browser (React components)
   │
   ├─ React Context (UI/session/editor төлөв) ──── [[E-Geree-v2-State-Management]]
   ├─ React Query (server cache)
   │
   ▼
Server Actions (src/lib/actions/*, "use server") ──── [[E-Geree-v2-Networking]]
   │
   ▼
Service модулиуд (src/lib/services/*)
   │
   ▼
httpRequest (src/lib/config/http-request) ── token, header нэмнэ
   │
   ▼
Гадаад микросервисүүд (AUTH_URL, BACKEND_URL, PAYMENT_URL, KYC_URL ...)
```

## Гол зарчмууд
- **Цэвэр frontend** — database/ORM байхгүй; бүх дата гадаад микросервисээс ирнэ.
- **3 давхаргат request flow** — Server Action → service → httpRequest. Клиент шууд
  гадаад backend руу хандахгүй. [[E-Geree-v2-Networking]]
- **Client/Server boundary** — RSC + `"use client"`. Auth-г `middleware.js` дээр
  JWT-ээр шалгаж `(protected)` route-уудыг хамгаална.
- **Олон хэл (i18n)** — бүх route `[locale]` segment дотор; next-intl-ээр. Хатуу
  бичсэн (hardcode) текст хориотой — `useTranslations('namespace')` ашиглана.
- **UI primitive заавал** — Radix-г шууд import хийхгүй, raw HTML биш, `components/ui/`.

## Дата урсгалын жишээ (гэрээ үүсгэх)
1. Wizard төлөв клиент дээр Context-д хадгалагдана ([[E-Geree-v2-Contract-Create-Feature]])
2. PDF талбарууд координаттай хадгалагдана ([[E-Geree-v2-PDF-Editor]])
3. Submit үед Server Action дуудаж service-ээр backend руу илгээнэ ([[E-Geree-v2-Networking]])
4. React Query хариуг cache-лж UI-г шинэчилнэ

## Граф-аас илэрсэн ажиглалт
- `useToast()` хамгийн их холбоостой (106 edge) — бараг бүх компонент дууддаг.
- `cn()` (Tailwind class util, 81 edge), `useTopLoaderRouter()` (68) — дундын util/nav.
- `useAuth()` (60), `useProfile()` (59), `usePayment()` (35) — гол context hook-ууд.
- `httpRequest` (41) — сүлжээний нэг цэг. HTTP verb helper-ууд (`getRequest`,
  `postRequest` г.м.) түүний эргэн тойронд нягт бүлэг (cohesion 0.30).
- Олон leaf компонент props биш, шууд context руу ханддаг тул эдгээр hook-ийн өөрчлөлт
  өргөн нөлөөтэй (blast radius их).
