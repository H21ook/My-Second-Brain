---
title: "Plan: Contract Detail Page — Phase 2a (detail route + data fetch + layout scaffold)"
type: project
status: draft
created: 2026-07-02
updated: 2026-07-02
tags:
  - project
  - project/e-geree-v3
  - plan
---

# Plan: Contract Detail Page — Phase 2a

**Огноо:** 2026-07-02
**Салбар:** dev-khishigee
**Төлөв:** 📝 Ноорог (батлагдаагүй)
**Холбоотой:** [[E-Geree-v3-Contract-Detail-Drawer-Phase1-Plan]] · [[E-Geree-v3-Networking-BFF]] · [[E-Geree-v3-State-Management]] · [[pdf-viewer-new]]

---

## Context (яагаад)

Phase 1 (drawer dropdown + rep modal + link/label/password modal + navigation) дуусаж commit хийгдсэн. Navigation одоо `/documents/detail/{id}/{actionId}` рүү үсэрдэг ч тэр route нь **stub** — зөвхөн `console.log(slug)` хийгээд `<div>ContractDetail</div>` рендэрлэдэг (`src/app/[locale]/(protected)/(with-sidebar)/documents/detail/[...slug]/page.tsx`).

Phase 2 бүхэлдээ том (detail page + rightSidebar accordion + ActionButtons + гарын үсэг/2FA/тоон гарын үсэг + PDF viewer + payment). Нэг plan-д багтахгүй тул **sub-phase болгон задалсан**. Энэ нь эхний зүсэм — **Phase 2a: detail route-ыг бодит уншиж-харах scaffold болгох**.

### Хамрах хүрээ (Phase 2a — энэ)
- `[...slug]` route-г бодит detail хуудас болгох (slug parse: `[contractId, actionId, accessCode]`).
- Client React Query-ээр detail data татах (`getContractDetail` + `useContractDetail`).
- `ContractDetail` layout: зүүн PDF viewer талбай + баруун SecondSidebar талбай — **хоёулаа placeholder**.
- Gate-ууд client-т: permission-denied (`!result`) + secured (accessCode gate).

### Хойшлуулсан (Phase 2b+ — энэ plan-д ОРОХГҮЙ)
- **2b:** SecondSidebar accordion (оролцогч / нэмэлт файл / түүх).
- **2c:** ActionButtons (sign/review/return/resend/cancel/accept-cancel/decline-cancel/pay).
- **2d:** Гарын үсэг зурах (signature-input) + 2FA/OTP (`SecurityVerificationModal`) + тоон гарын үсэг (`DigitalSignature`).
- **PDF viewer бодит холболт:** [[pdf-viewer-new]] тусдаа effort — энэ scaffold зөвхөн hook үлдээнэ.
- **Payment:** `getPaymentByContractRequest` урсгал.

---

## Архитектурын шийдвэрүүд (батлагдсан)

| Шийдвэр | Сонголт | Учир |
|---|---|---|
| Data fetch | **Client React Query** | v3-ийн тогтсон pattern (received list, contract-create). v2-ийн server-action-аас ялгаатай — v3 documents бүгд client. |
| Route param | **`[...slug]` хэвээр, triple parse** | `[contractId, actionId, accessCode]` — v2-той ижил. Phase 1 navigation аль хэдийн `/detail/{id}/{actionId}` руу үсэрдэг тул required catch-all хангалттай. |
| PDF viewer | **Placeholder stub + hook** | pdf-viewer-new тусдаа effort. 2a-г тэр scope-той холихгүй. |
| Gate | **Client-т** | v2-ийн server gate-ийг client component-т буулгана (permission-denied + secured). |

---

## v3 дээр бэлэн, дахин ашиглах хэсгүүд

| Хэрэгцээ | Бэлэн эх сурвалж | Замд |
|---|---|---|
| React Query + apiClient BFF загвар | Phase 1 documents api | `src/features/documents/api/{client,queries}.ts` |
| Browser HTTP client (envelope unwrap) | `apiClient` (axios, `baseURL:"/backend"`) | `src/core/http/api-client.ts` |
| Detail data татах загвар | `ReceivedContent` inline `useQuery` (L134-168) | `src/features/documents/components/ReceivedContent.tsx` |
| Contract type | `ContractListItem` (detail талбар нэмэх шаардлагатай) | `src/types/contract-types.ts` |
| Loading/skeleton | shadcn `skeleton.tsx` | `src/components/ui/skeleton.tsx` |
| Access-code input | Phase 1 `PasswordChangeModal` загвар + `input-otp` | `src/features/documents/components/` |
| i18n navigation | `useRouter`, `Link` | `@/i18n/navigation` |

---

## Ажлын зүйлс (Phase 2a)

### 1. Detail type + `getContractDetail` — `documents/api/{client,queries}.ts`
`ContractListItem`-ийг detail талбараар өргөтгөх (эсвэл `ContractDetail` type тусад нь): `actionList`, `contractFieldList`, `config.participantsConfig`, `signedPdfUrl`/`generatedPdfUrl`/`relatedPdfUrl`, `secure`, `paymentType`/`paymentStatus`.

```
getContractDetail(contractRequestId, actionId, accessCode?)
  → GET /public-api/v1/contract-request?contractRequestId={id}&actionId={actionId}[&accessCode=...]
```

> **Backend/version verify:** v2-т `BACKEND_URL` (v1 base) дээр. accessCode нь илгээхээс өмнө v2-т `decryptPassword` хийгддэг — v3-т энэ decrypt хэрэгтэй эсэхийг backend-тэй тулгаж баталгаажуулах. GET тул BFF okNoop асуудалгүй. Дэлгэрэнгүй: [[E-Geree-v3-Networking-BFF]].

`useContractDetail(contractId, actionId, accessCode?)` — `useQuery`, key factory `documentsKeys.detail(id, actionId)`, `enabled: Boolean(contractId && actionId)`.

### 2. Route wrapper — `documents/detail/[...slug]/page.tsx`
Одоогийн stub-г солино. Server component slug parse хийж client component руу дамжуулна (эсвэл бүхэлд нь client):
```
const [contractId, actionId, accessCode] = slug ?? [];
→ <ContractDetail contractId=… actionId=… accessCode=… />
```

### 3. `ContractDetail` layout — `documents/components/ContractDetail.tsx` (шинэ, client)
- `useContractDetail` дуудна.
- **Loading:** skeleton (2 багана).
- **Error / `!result`:** permission-denied UI (v2 `PermissionDenied` загвар — түгжээ icon + "жагсаалт руу буцах" товч).
- **`secure` && accessCode алга:** `<SecuredDetailContent>` рендэр (4-р зүйл).
- **Амжилт:** 2 баганат layout — зүүн `<ViewerPlaceholder>`, баруун `<SidebarPlaceholder>`.

### 4. `SecuredDetailContent` — accessCode gate (шинэ, client)
- accessCode оруулах input (`input-otp` эсвэл password input) + "Нээх" товч.
- Оруулсан код → `useContractDetail(…, accessCode)` refetch. Буруу бол алдаа харуулна.
- v2 эх сурвалж: `contract-detail/secured-page`.

### 5. Placeholder компонентууд (2b/pdf-viewer hook)
- **`ViewerPlaceholder`** — "PDF viewer энд холбогдоно" skeleton. `pdfViewUrl` (signed/generated/related) тооцоолол энд бэлдэж, [[pdf-viewer-new]]-д props болгож дамжуулах цэг үлдээнэ.
- **`SidebarPlaceholder`** — accordion сав (оролцогч/файл/түүх гарчигтай, хоосон биетэй). 2b-д гүйцээнэ.

---

## Endpoint лавлагаа (v2-ээс)

| Үйлдэл | Method | Path (v2) | v2 backend const |
|---|---|---|---|
| Гэрээний дэлгэрэнгүй | GET | `/contract-request?contractRequestId={id}&actionId={actionId}[&accessCode=]` | `BACKEND_URL` |
| Нэмэлт файл (2b) | GET | `getAdditionalFileList(contractRequestId)` | — |
| Payment (later) | GET | `getPaymentByContractRequest(id)` | — |

---

## Verification (Phase 2a)

1. **Build/lint/knip:** `npm run lint && npm run knip && npx tsc --noEmit` — placeholder-ууд unused export үлдээхгүй.
2. **Navigation:** received жагсаалт → drawer → "Дэлгэрэнгүй харах" (эсвэл rep-modal сонголт) → `/documents/detail/{id}/{actionId}` → detail хуудас ачаалж, layout (2 багана placeholder) харагдана.
3. **Loading:** удаан сүлжээнд skeleton гарна.
4. **Secured:** `secure` гэрээ дээр accessCode gate гарч, зөв код оруулахад detail ачаална.
5. **Permission-denied:** буруу эрх/буруу id дээр permission-denied UI + жагсаалт руу буцах товч.
6. **E2E:** `npm run test:e2e` (critical-flows эвдрээгүй).
7. Гар аргаар `npm run dev`: дээрх урсгалуудыг гүйцэтгэх.

---

## Файлын товч жагсаалт

**Шинэ:**
- `src/features/documents/components/ContractDetail.tsx`
- `src/features/documents/components/SecuredDetailContent.tsx`
- `src/features/documents/components/detail/ViewerPlaceholder.tsx`, `SidebarPlaceholder.tsx`

**Засах:**
- `src/app/[locale]/(protected)/(with-sidebar)/documents/detail/[...slug]/page.tsx` (stub → wrapper)
- `src/features/documents/api/client.ts` (`getContractDetail` + detail type)
- `src/features/documents/api/queries.ts` (`useContractDetail` + `documentsKeys.detail`)
- `src/types/contract-types.ts` (detail талбарууд)

---

## v2 эх сурвалжийн лавлагаа (порт хийхэд)

- Detail page урсгал: `e-geree-v2/.../documents/(my-documents)/detail/[[...slug]]/page.jsx` (156 мөр)
- Detail data action: `e-geree-v2/src/lib/actions/contract.js` `getContractDetail` (L41)
- ContractDetail layout: `e-geree-v2/src/components/pages/contract-detail/`
- Secured page: `e-geree-v2/src/components/pages/contract-detail/secured-page`
- SecondSidebar (2b): `e-geree-v2/.../documents/second-sidebar/index.jsx` (642 мөр) — accordion item-ууд тэр директорт
- PermissionDenied: `e-geree-v2/.../documents/permission-denied/index.jsx`

---

## Гүйцэтгэлийн дараалал, модел, токен хэмнэлт

### Дараалал (алхам алхмаар)

| #    | Алхам                                                   | Файл                                                     | Хамаарал | Санал болгох модел               |
| ---- | ------------------------------------------------------- | -------------------------------------------------------- | -------- | -------------------------------- |
| 2a-1 | Detail type + `getContractDetail` + `useContractDetail` | `documents/api/{client,queries}.ts`, `contract-types.ts` | —        | Sonnet 5                         |
| 2a-2 | Route wrapper (stub → slug parse)                       | `detail/[...slug]/page.tsx`                              | 2a-1     | Haiku 4.5 (trivial)              |
| 2a-3 | `ContractDetail` layout + gate + loading/error          | `ContractDetail.tsx`                                     | 2a-1     | **Opus 4.8** (gate логик нарийн) |
| 2a-4 | `SecuredDetailContent` accessCode gate                  | `SecuredDetailContent.tsx`                               | 2a-1     | Sonnet 5                         |
| 2a-5 | Viewer/Sidebar placeholder                              | `detail/*Placeholder.tsx`                                | —        | Haiku 4.5 (trivial)              |
| ✅    | **2a verify + commit** (verification 1–6)               | —                                                        | —        | —                                |
| —    | Phase 2b (sidebar accordion)                            | тусдаа plan/чат                                          | —        | Sonnet 5                         |
| —    | Phase 2c (ActionButtons)                                | тусдаа plan/чат                                          | —        | Opus (нөхцөлт логик)             |
| —    | Phase 2d (гарын үсэг + 2FA + тоон гарын үсэг)           | тусдаа plan/чат                                          | —        | Opus (архитектур)                |

Ерөнхий дүрэм: **2a бүрэн (verify+commit) → дараа нь 2b**. Алхам бүр = жижиг diff = 1 commit.

### Effort түвшин

| Алхам төрөл | Алхмууд | Effort |
|---|---|---|
| Механик / trivial (route parse, placeholder) | 2a-2, 2a-5 | **low** |
| Api модуль, type өргөтгөл, query wiring | 2a-1, 2a-4 | **medium** |
| Gate/загварын салаа логик (loading/error/secured/success) | 2a-3 | **high** |
| Phase 2d архитектур (signing + 2FA) | 2d | **xhigh** |

### Чат стратеги (токен хэмнэх)
- **2a — шинэ чат.** Seed: энэ plan note + target файлууд. Хуучин explore үүрэхгүй.
- Seed prompt загвар: *"Read plan note: `<зам>`. Implement алхам 2a-1 зөвхөн. Read зөвхөн: `documents/api/{client,queries}.ts`, `contract-types.ts`, v2 `getContractDetail`."*
- **2b/2c/2d — фаза бүрд ШИНЭ чат + тусдаа plan note.**

### Токен-бага, чанар-өндөр горим
1. Дахин explore хийхгүй — plan note бол эх сурвалж; бүх зам/endpoint дотор нь бий.
2. Зорилтот файл л унших — алхмын хүснэгтэд заасан төдий.
3. **graphify эхэлж** (repo дүрэм): `graphify query "..."` → дараа нь тодорхой мөр засах/унших.
4. Тусгаарлагдсан механик порт → `cavecrew-builder` subagent.
5. Алхам бүрийн дараа: тухайн файлд `tsc --noEmit` + `eslint` → commit.
6. `knip`-ийг 2a төгсгөлд л ажиллуулна.
7. caveman + ponytail асаалттай хэвээр (богино diff, богино тайлбар).

---

## Гүйцэтгэлийн явц (progress log)

### ✅ Phase 2a — ДУУССАН (commit 4f96511, 2026-07-02)
- 2a-1 Detail type + `getContractDetail` + `useContractDetail` ✅
- 2a-2 Route wrapper (slug parse) ✅
- 2a-3 `ContractDetail` layout + gate (loading/error/secured/success) ✅
- 2a-4 `SecuredDetailContent` accessCode gate ✅
- 2a-5 Viewer/Sidebar placeholder ✅

### ✅ Phase 2b — SecondSidebar accordion — ДУУССАН (2026-07-03, энэ чат)
- **Type:** `ContractHistory`, `AdditionalFile` interface; `ContractDetailData.historyList?` нэмсэн — `contract-types.ts`.
- **API:** `getAdditionalFileList(contractId)` → `GET /public-api/v1/contract-addition-file/list?contractId=..&type=SENT` — `api/client.ts`.
- **Hook:** `useAdditionalFiles(contractId)` + `documentsKeys.additionalFiles` — `api/queries.ts`.
- **UI:** `SidebarPlaceholder` → `ContractSidebar.tsx` (3 хэсэг: оролцогч / нэмэлт файл / түүх). Нэмэлт файл loading+empty state; түүх historyList render.
- **♻️ Refactor:** оролцогчийн accordion-г `ParticipantAccordion.tsx` болгож drawer-тэй **дундалдсан** (нэг эх). `CopyIconButton` тэнд шилжсэн (export). Drawer 140 мөр inline block + local const устгасан. Switch-user товч `canSwitchUser`/`onSwitchUser` prop-оор (drawer л дамжуулна).
- **Verify:** `tsc --noEmit` 0 error, `eslint` цэвэр (үлдсэн 2 warning = pre-existing Trash2/KeyRound), `knip` цэвэр.

**Хойшлуулсан (2b-д ОРООГҮЙ):** файл upload/delete/rename/download + 2FA → 2d; оролцогчийн resend/distribute → 2c.

**⚠️ Backend-тэй тулгах өр:**
1. `contract-addition-file` endpoint prefix (`/public-api/v1` таамаг) — 404 бол v2 руу.
2. `historyList` v1 detail хариунд ирдэг эсэх (v2-т `contractDetail.historyList`) — байхгүй бол accordion хоосон.
3. `accessCode` encrypt хэрэгтэй эсэх (2a-аас үлдсэн, 2d-д шийднэ).

### ⬜ Дараагийн — [[E-Geree-v3-Contract-Detail-Page-Phase2c-Plan]] ✅ (ActionButtons) → [[E-Geree-v3-Contract-Detail-Page-Phase2d-Plan]] (sign/2FA/digital) → [[pdf-viewer-new]]
