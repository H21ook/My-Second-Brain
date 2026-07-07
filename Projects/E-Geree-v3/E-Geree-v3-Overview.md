---
title: "01 — Төслийн тойм"
type: project
status: active
created: 2026-06-30
updated: 2026-07-07
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/01-Overview.md"
---

# Төслийн тойм

[[E-Geree-v3-Home]] · дараах: [[E-Geree-v3-Architecture]]

## Юу вэ
**E-Geree-v3** — цахим гэрээний (e-contract) платформын frontend. Хэрэглэгч гэрээний
PDF байршуулж, оролцогчдыг тодорхойлж, PDF дээр талбар (гарын үсэг, огноо, текст г.м.)
байрлуулаад, гэрээг илгээж гарын үсэг цуглуулдаг.

## Технологийн стек
| Давхарга | Технологи |
|---|---|
| Framework | Next.js 16 (App Router), React 19 |
| Хэл | TypeScript |
| Төлөв (state) | Redux Toolkit + redux-persist |
| Сервер дата | TanStack React Query |
| Сүлжээ | Axios, дотоод BFF (route handlers) |
| Validation | Zod + React Hook Form |
| Хүснэгт | TanStack React Table |
| Олон хэл | next-intl (mn, en, kr, cn) |
| PDF | pdfjs-dist v5 |
| UI | shadcn/ui (Radix/base-ui + Tailwind) |
| Socket | socket.io-client `^4.8.3` — GSIGN тоон гарын үсгийн push (`features/documents/components/detail/DigitalSignModal.tsx`, upstream: `core/config/index.ts:52` `getDigitalSignatureUrl`, BFF: `app/backend/digital-signature/[...path]/route.ts`) |
| Тест | Vitest `^4.1.9` (unit — 475 тест / 12 файл, 2026-07-07 байдлаар) + Playwright (E2E) |

## Дээд түвшний хавтас (src/)

- `app/` — Next.js App Router (routes + BFF). [[E-Geree-v3-Routing]]
- `features/` — Vertical slice feature-ууд: `contract-create/`, `documents/`, `profile/`. [[E-Geree-v3-Contract-Create-Feature]] · [[E-Geree-v3-Contract-Detail]] · [[E-Geree-v3-Profile-Migration-Plan]]
- `core/` — Үндсэн дэд бүтэц: `http/` (fetcher), `config/`, `auth/`, `observability/`. [[E-Geree-v3-Networking-BFF]]
- `shared/` — Дундын давхарга: `ui/components/`, `lib/`, `pdf-viewer/`. [[E-Geree-v3-PDF-Viewer]]
- `components/` — UI: `ui/` (shadcn primitive), `custom/`, `layout/`, `auth/`
- `store/` — Глобал Redux store (auth). [[E-Geree-v3-State-Management]]
- `lib/` — Утилит: `schemas/`, `services/`, `constants.ts`, `bff-observability.ts`, `utils.ts` (`cn`)
- `types/` — Глобал TS төрлүүд (auth, company, contract, enums)
- `providers/` — React provider-ууд (query, store, theme)
- `hooks/` — Дундын hook-ууд (use-debounce, use-mobile)
- `actions/` — Server actions (auth-actions)
- `i18n/` — next-intl тохиргоо + хэлний өгөгдөл

## npm скрипт
- `dev` — хөгжүүлэлт · `dev:e2e` — mock сервертэй · `build` / `start`
- `lint` · `knip` · `test` (Vitest) / `test:watch` · `test:e2e` (Playwright, мөн `:ui` / `:headed`)
