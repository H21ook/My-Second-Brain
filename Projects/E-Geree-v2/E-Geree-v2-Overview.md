---
title: "01 — Төслийн тойм"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v2
source_path: "D:/own/obsidian-vaults/E-Geree-v2-docs/01-Overview.md"
---

# Төслийн тойм

[[E-Geree-v2-Home]] · дараах: [[E-Geree-v2-Architecture]]

## Юу вэ
**E-Geree-v2** — цахим гэрээний (e-contract) платформын frontend. Хэрэглэгч гэрээний
PDF байршуулж, оролцогчдыг тодорхойлж, PDF дээр талбар (гарын үсэг, огноо, текст г.м.)
байрлуулаад, гэрээг илгээж гарын үсэг цуглуулдаг.

## Технологийн стек
| Давхарга | Технологи |
|---|---|
| Framework | Next.js 14 (App Router) |
| Хэл | JavaScript (TS биш — бүх route `.jsx`) |
| Төлөв (state) | React Context (21 provider) |
| Сервер дата | TanStack React Query |
| Сүлжээ | fetch wrapper + Server Actions, гадаад микросервисүүд |
| Олон хэл | next-intl (mn default, en, ko, zh) |
| PDF | react-dnd дээр суурилсан өөрийн editor |
| UI | Radix UI + class-variance-authority + Tailwind |
| Observability | OpenTelemetry (`ENABLE_INSTRUMENTATION=true` үед) |
| Тест | Jest (integration) |

## Архитектурын онцлог
Дотроо **database/ORM байхгүй** — цэвэр frontend. Бүх дата гадаад микросервисээс
HTTP/WebSocket-ээр ирнэ. Backend URL-ууд environment хувьсагч: `AUTH_URL`,
`BACKEND_URL`, `BACKEND_URL_V2`, `MANAGE_URL`, `SERVICE_URL`, `DIGITAL_SIGNATURE_URL`,
`PAYMENT_URL`, `KYC_URL`.

## Дээд түвшний хавтас (src/)
- `app/[locale]/` — Next.js App Router (routes). [[E-Geree-v2-Routing]]
- `components/pages/<feature>/` — Хуудас бүрийн гол компонент
- `components/context/` — 21 React Context provider. [[E-Geree-v2-State-Management]]
- `components/ui/` — Radix дээрх UI primitive-ууд (Button, Dialog, Select г.м.)
- `components/custom/pdf/` — PDF гэрээ засварлагч. [[E-Geree-v2-PDF-Editor]]
- `lib/actions/` — Server Actions ("use server"). [[E-Geree-v2-Networking]]
- `lib/services/` — Domain service модулиуд (httpRequest wrapper)
- `lib/config/http-request/` — Доод түвшний fetch wrapper
- `utils/` — `enums.js`, `index.js`, `static-data.js`, `cookie-utils.js`
- `i18n/` — next-intl message файлууд
- `middleware.js` — locale routing + JWT хамгаалалт

## npm скрипт
- `dev` — хөгжүүлэлт (port 4002) · `dev-ins` — OTEL-тэй
- `build` — production · `lint` — ESLint
- `test` — Jest · `build:test` — build + integration тест
