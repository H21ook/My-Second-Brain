---
title: "02 — Архитектур"
type: project
status: active
created: 2026-06-30
updated: 2026-07-07
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
   │  · auth(+auth-v2) proxy, file upload, public-api v1/v2,
   │    payment(+payment-v2), digital-signature
   ▼
Гадаад backend API
```

> Үл хамаарах цорын ганц зам: digital-signature-ийн **socket** холболт BFF-ээр
> дамждаггүй — client шууд socket сервер рүү холбогдоно (доорх
> "Модулийн хариуцлага" хэсгийг үз).

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
    backend/{auth,auth-v2,public-api,file,health,payment,payment-v2,digital-signature}/
                                          # BFF proxy → гадаад API (бүгд [...path] catch-all;
                                          # auth нь нэмээд static route-уудтай: check-auth,
                                          # login-by-email, logout, refresh-token,
                                          # user/{merge-account,upload-user-image,verify-email-confirmation})
  proxy.ts                              # middleware: locale + auth gating + token refresh

  features/                             # Vertical slice — тус бүр public index.ts-тэй
    contract-create/  {api, components/steps, config, hooks, lib, schema, store, types, index.ts}
    documents/        {api, components/detail, lib, index.ts}  → [[E-Geree-v3-Contract-Detail]]
    profile/          {api, components, config, hooks, lib, schema, index.ts}  # useTeamList · useCompanyEmployees · useEmployeePage г.м

  shared/                               # Cross-feature, domain логикгүй
    pdf-viewer/  {*.tsx, hooks/, store/, utils/, types.ts, index.ts}  → [[E-Geree-v3-PDF-Viewer]]
    ui/components/                      # App-level composite UI
    lib/         custom-utils.ts

  core/                                 # Platform / server суурь  → [[E-Geree-v3-Networking-BFF]]
    http/          core-fetcher · server-fetcher · backend-handler · headers · api-client
    config/        getAuthUrl(V2) · getBackendUrl(V2) · getPaymentUrl(V2) · getDigitalSignatureUrl
    auth/          tokens · cookie-options
    observability/ logger

  components/                           # Хараахан feature/shared болоогүй үлдэгдэл
    ui/      shadcn primitive kit (~55) — зориудаар байрандаа
    custom/  home/ · login/
    auth/    auth-initializer · global-profile-loader
    layout/  app-sidebar · nav-* (nav-labels ← label sidebar) · ShareLabelDialog
             · CreateDocumentDialog · dynamic-breadcrumb · team-switcher

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
  (Цорын ганц үл хамаарал: digital-signature-ийн socket холболт — доор.)
- **Client/Server boundary** — RSC (server component) + `"use client"` хослол.
  Auth-г server дээр шалгаж, layout түвшинд хамгаална.
- **Олон хэл (i18n)** — бүх route `[locale]` segment дотор; next-intl-ээр.

## Давхаргын дүрэм (enforced)
Import чиглэлийн ерөнхий зарчим: **`app → features → shared → core`**.
Бодит enforcement нь `eslint.config.mjs`-ийн `boundaries/element-types` дүрэм
(default = **allow**, зөвхөн 2 хориг, `eslint.config.mjs:68-90`):

| from давхарга | Хориотой import | Баталгаа |
|---|---|---|
| `feature` (`src/features/<x>`) | өөр `features/<y>`-ийн **бүх** файл — public `index.ts`-г **оролцуулаад** | `eslint.config.mjs:74-80` — алдааны мессеж index.ts-г санал болгодог ч disallow pattern `src/features/*` бүх файлд таардаг |
| `shared-ui` (`components/ui`, `shared/ui`) · `lib` (`src/lib`, `shared/lib`) · `shared` (`src/shared/*`) | `feature`, `app`, `app-component` | `eslint.config.mjs:81-88` |
| `app` · `app-component` (`components/{custom,auth,layout}`) · `store` · `platform` (i18n/hooks/providers/types/actions) | — хориггүй (default allow) | ж: `nav-labels.tsx` (app-component) `features/documents`-ийн дотоод файлуудыг шууд импортолдог — lint OK |

- **app-component → feature ЗӨВШӨӨРНӨ** — тиймээс cross-feature состав хэрэгтэй
  бол `components/layout/`-д угсарна (ж: `ShareLabelDialog.tsx` — `features/profile`
  + `features/documents` хоёуланг ашигладаг тул layout-д байрласан).
- `src/core`-д boundaries element-type **байхгүй** — `core → shared/lib` чиглэл нь
  конвенц, lint-ээр enforce хийгддэггүй.
- **Boundary lint** — `eslint-plugin-boundaries` дээрх 2 invariant-г **error** болгоно.
- **Файл нэрлэлт** — компонент `PascalCase`, hook `camelCase` (`use*`), бусад модуль
  `kebab-case`; `eslint-plugin-check-file`-ээр баталгаажна.
- **Guardrail** — `knip` (dead-code CI) + Vitest unit-test runner (Playwright E2E-ийн доор).

## Дата урсгалын жишээ (гэрээ үүсгэх)
1. Wizard slice-ууд (participants/content/fields/meta) клиент дээр төлөв хадгална
2. redux-persist-ээр localStorage-д хадгалж, дахин ачаалахад сэргээнэ
3. PDF талбарууд store-д координаттай хадгалагдана ([[E-Geree-v3-PDF-Viewer]])
4. Submit үед payload угсарч ([[E-Geree-v3-Contract-Create-Feature]]) BFF-ээр илгээнэ

## Модулийн хариуцлага (гол шинэ модулиуд)

### Тоон гарын үсэг (digital-signature, GSIGN)
- `useStartGSign` (`src/features/documents/api/queries.ts:90`) — create-sign-request
  + g-sign push-ийг нэг мутацид нэгтгэсэн; буцаах утга = socket room key
  (`pdfSignRequestId`). Invalidate энд хийхгүй — эцсийн баталгаа socket-оор ирдэг.
- `DigitalSignModal` (`src/features/documents/components/detail/DigitalSignModal.tsx`) —
  утас → push → socket хүлээлт UI. `socket.io-client@^4`-ээр
  `NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL`-ийн `/digital-signature` namespace руу
  **шууд** холбогдоно — энэ socket нь BFF-г тойрдог цорын ганц client холболт.
- REST тал нь BFF-ээр: `/backend/digital-signature/[...path]` →
  `getDigitalSignatureUrl()` (`src/core/config/index.ts:52`).
  Дэлгэрэнгүй: [[E-Geree-v3-Contract-Detail]].

### Label sidebar (NavLabels)
- `src/components/layout/nav-labels.tsx` — `useLabelList` мод (Collapsible tree),
  CRUD dialog-ууд `features/documents/components/`-оос, drag&drop drop-target
  (native HTML5 DnD, `features/documents/lib/drag.ts`-ийн `CONTRACT_DRAG_TYPE`).
- Хуваалцах: `src/components/layout/ShareLabelDialog.tsx` — feature→feature хориг
  тул layout давхаргад; `features/profile`-ийн `useTeamList`/`useEmployeePage`-г
  ашиглана. Дэлгэрэнгүй: [[E-Geree-v3-Label-Sidebar]].

## Граф-аас илэрсэн ажиглалт
- `cn()` (Tailwind class util) хамгийн их холбоостой — бараг бүх UI компонент дууддаг.
- `useAppSelector` / `useAppDispatch` — Redux хандалтын гол цэг.
- Domain модель: `LegalEntityType`, `ParticipantConfig`, `Participant`, `ContractField`.
