---
title: "02 — Архитектур"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/02-Architecture.md"
---

# Архитектур

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-Overview]] · дараах: [[E-Geree-v3-Routing]]

## Давхаргат бүтэц
```
Browser (React components)
   │
   ├─ Redux store (client state) ──── [[E-Geree-v3-State-Management]]
   ├─ React Query (server cache)
   │
   ▼
Next.js BFF (route handlers, src/app/backend/*) ──── [[E-Geree-v3-Networking-BFF]]
   │  · auth proxy, file upload, public-api v1/v2
   ▼
Гадаад backend API
```

## Хавтасны бүтэц (src/)
> Refactor (`refactor/architecture-cleanup`)-ийн дараах бодит бүтэц. Шинэ давхаргууд:
> `features/`, `shared/`, `core/`. `components/*`, `actions/`, `hooks/`, `providers/` нь
> түүхэн шалтгаанаар дээд түвшинд үлдсэн (зориудаар, цаашид аажмаар нүүлгэх боломжтой).

```
src/
  app/                                  # App Router — routing + composition (бизнес логикгүй)
    [locale]/
      (public)/ (auth)/ (protected)/{(with-sidebar)/,(without-sidebar)/}
      layout.tsx  error.tsx  not-found.tsx  og-image.tsx
    backend/{auth,public-api,file,health}/   # BFF proxy → гадаад API
  proxy.ts                              # middleware: locale + auth gating + token refresh

  features/                             # Vertical slice — тус бүр public index.ts-тэй
    contract-create/  {api, components/steps, config, hooks, lib, schema, store, types, index.ts}
    documents/        {components, index.ts}

  shared/                               # Cross-feature, domain логикгүй
    pdf-viewer/  {*.tsx, hooks/, store/, utils/, types.ts, index.ts}  → [[E-Geree-v3-PDF-Viewer]]
    ui/components/                      # App-level composite UI
    lib/         custom-utils.ts

  core/                                 # Platform / server суурь  → [[E-Geree-v3-Networking-BFF]]
    http/          core-fetcher · server-fetcher · backend-handler · headers · api-client
    config/        getAuthUrl / getBackendUrl(V2)
    auth/          tokens · cookie-options
    observability/ logger

  components/                           # Хараахан feature/shared болоогүй үлдэгдэл
    ui/      shadcn primitive kit (~55) — зориудаар байрандаа
    custom/  home/ · login/
    auth/    auth-initializer · global-profile-loader
    layout/  app-sidebar · nav-* · breadcrumb · team-switcher

  store/      root Redux (auth-slice + index)  → [[E-Geree-v3-State-Management]]
  lib/        utils(cn)+test · constants · bff-observability · schemas/ · services/
  actions/    auth-actions.ts           # цорын ганц server action
  hooks/      useDebounce · useMobile
  providers/  query / store / theme
  i18n/       тохиргоо + languages-data/
  types/      глобал TS төрлүүд
```

## Гол зарчмууд
- **Feature-based бүтэц** — `features/contract-create/` дотроо өөрийн store,
  api, hooks, lib, config, schema, components-тэй бие даасан модуль.
- **BFF давхарга** — клиент гадаад backend руу шууд хандахгүй; `src/app/backend/*`
  route handler-ууд токен, cookie, observability-г төвлөрүүлж зуучилна.
- **Client/Server boundary** — RSC (server component) + `"use client"` хослол.
  Auth-г server дээр шалгаж, layout түвшинд хамгаална.
- **Олон хэл (i18n)** — бүх route `[locale]` segment дотор; next-intl-ээр.

## Давхаргын дүрэм (enforced)
Import чиглэл: **`app → features → shared → core`**. Feature-ууд бие биенийхээ
дотоод файлыг **импортлохгүй** — зөвхөн `features/<x>/index.ts` нийтийн surface-аар.

| Давхарга | Импортлож болох | Импортлож болохгүй |
|---|---|---|
| `app/` | features (index), shared, core | өөр feature-ийн дотоод |
| `features/<x>` | shared, core, өөрийн дотоод | өөр `features/<y>` дотоод |
| `shared/*` | shared/lib, shared/ui | features, app |
| `core/` | shared/lib | features, app |

- **Boundary lint** — `eslint-plugin-boundaries` дээрх 2 invariant-г **error** болгоно.
- **Файл нэрлэлт** — компонент `PascalCase`, hook `camelCase` (`use*`), бусад модуль
  `kebab-case`; `eslint-plugin-check-file`-ээр баталгаажна.
- **Guardrail** — `knip` (dead-code CI) + Vitest unit-test runner (Playwright E2E-ийн доор).

## Дата урсгалын жишээ (гэрээ үүсгэх)
1. Wizard slice-ууд (participants/content/fields/meta) клиент дээр төлөв хадгална
2. redux-persist-ээр localStorage-д хадгалж, дахин ачаалахад сэргээнэ
3. PDF талбарууд store-д координаттай хадгалагдана ([[E-Geree-v3-PDF-Viewer]])
4. Submit үед payload угсарч ([[E-Geree-v3-Contract-Create-Feature]]) BFF-ээр илгээнэ

## Граф-аас илэрсэн ажиглалт
- `cn()` (Tailwind class util) хамгийн их холбоостой — бараг бүх UI компонент дууддаг.
- `useAppSelector` / `useAppDispatch` — Redux хандалтын гол цэг.
- Domain модель: `LegalEntityType`, `ParticipantConfig`, `Participant`, `ContractField`.
