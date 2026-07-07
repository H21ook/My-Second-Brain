---
title: "Plan: Contract Detail Page — Phase 2c (ActionButtons — гэрээний үйлдлүүд)"
type: project
status: in-progress
created: 2026-07-03
updated: 2026-07-07
tags:
  - project
  - project/e-geree-v3
  - plan
---

# Plan: Contract Detail Page — Phase 2c (ActionButtons)

> [!warning] Аудит 2026-07-07 (Claude) — дутуу зүйлс
> **Ерөнхий төлөв:** mostly-done (9/11 хэрэгжсэн)
> **Дутуу / хийгдээгүй:**
> - 🟡 Verification алхам 2-4: browser/live шалгалт — 3 reason action (return/cancel-request/decline-cancellation), товчны enable/disable matrix, delete→confirm→жагсаалт урсгал — Worklog 2026-07-03 "Not verified in-browser"; 2d-ийн live session-ууд зөвхөн approve OTP + fill/signature-ийг хамарсан тул "backend prefix тулгах" өрийн payload/path батлагдаагүй хэвээр.
> - ⚪ Verification алхам 7: `npm run test:e2e` (Playwright) ажиллуулсан бичлэг алга — test-results/.last-run.json 2026-06-30 (2c commit-оос өмнө), сүүлийн run улаан (5 тест унасан).
> **Тэмдэглэл:** Бүх build ажил commit `7d4afe2`-т хийгдсэн (3 endpoint + 3 mutation hook, ActionButtons.tsx, FeedbackModal.tsx, ConfirmActionDialog; tsc/eslint/knip цэвэр); sign/pay placeholder-ууд 2d-д бодит болсон. Frontmatter `status: draft` хуучирсан — шинэчлэх, мөн 3 reason action-ы live шалгалт + e2e run хийхээр шийдэх хэрэгтэй.

**Огноо:** 2026-07-03
**Салбар:** dev-khishigee
**Төлөв:** 📝 Ноорог (батлагдаагүй)
**Холбоотой:** [[E-Geree-v3-Contract-Detail]] (2a plan устгагдсан) · [[E-Geree-v3-Networking-BFF]] · [[E-Geree-v3-State-Management]]

---

## Context (яагаад)

Phase 2a (detail route + gate + layout) ✅ commit `4f96511`. Phase 2b (sidebar accordion — оролцогч/файл/түүх + `ParticipantAccordion` refactor) ✅ commit `7a533e7`. Detail хуудас одоо **зөвхөн харах** — үйлдэл алга.

Phase 2c = гэрээ дээрх **үйлдлийн товчнууд (ActionButtons)**: буцаах / дахин илгээх / хянах / зөвшөөрөх / цуцлах хүсэлт / цуцлалт хүлээн авах-татгалзах / устгах. Эдгээр нь **энгийн POST үйлдэл** (2FA/гарын үсэг шаардахгүй). Гарын үсэг зурах, тоон гарын үсэг, төлбөр нь 2d/payment-т хамааралтай тул энд **зөвхөн placeholder handler** үлдээнэ.

### Хамрах хүрээ (Phase 2c — энэ)
- `ActionButtons` компонент (presentational) — товчны идэвх status/permission-оос тооцоолно.
- Энгийн үйлдлийн mutation hook-ууд: return / resend / review / approve / cancel-request / accept-cancellation / decline-cancellation / delete.
- Амжилтад detail query invalidate + жагсаалт руу / refetch.
- Баталгаажуулах dialog (устгах/цуцлах) — Phase 1 `DeleteConfirmDialog` загвар.

### Хойшлуулсан (2d+ — ОРОХГҮЙ)
- **Sign / Digital sign / 2FA(OTP/DAN):** `SecurityVerificationModal` + signature-input + `DigitalSignature` → **2d**. 2c-д sign товч зөвхөн `onSign()` placeholder дуудна.
- **Pay:** `getPaymentByContractRequest` + төлбөрийн урсгал → payment effort.
- **VAT modal-ууд** (`handleShowVatModal/Create`) → payment/2d.
- **PDF viewer** → `pdf-viewer-new`.

---

## Архитектурын шийдвэрүүд

| Шийдвэр | Сонголт | Учир |
|---|---|---|
| ActionButtons бүтэц | **Presentational + props/handler** | v2-той ижил — enable-логик props-оос; API дуудлага hook-т. Тест/дахин ашиглахад амар. |
| Action mutation | **React Query `useMutation`** | v3 pattern (`useDeleteContract`, `useLinkContract`). onSuccess → `documentsKeys.detail` invalidate. |
| Товч enable | **`contractStatus` + `actionStatus` + `permissionType`-оос** | v2 `contract-action-buttons` логик 1:1 порт. |
| Sign/digital/pay | **Placeholder handler (2c), бодит 2d** | 2c-г signing scope-той холихгүй. |
| Confirm dialog | **`DeleteConfirmDialog` (Phase 1)** дахин ашиглах | Delete/cancel-д. |

---

## v3 дээр бэлэн, дахин ашиглах

| Хэрэгцээ | Эх сурвалж | Замд |
|---|---|---|
| Mutation + invalidate загвар | `useDeleteContract`, `useLinkContract` | `features/documents/api/queries.ts` |
| Confirm dialog | `DeleteConfirmDialog` | `features/documents/components/DeleteConfirmDialog.tsx` |
| Button / Tooltip primitive | shadcn | `components/ui/{button,tooltip}.tsx` |
| Одоогийн profile (эрх шалгах) | `useAppSelector(state.auth.selectedProfile)` | `@/store` |
| Status enum-ууд | `ActionStatus`, `ContractStatusEnum` | `types/enums.ts` |
| Detail data (actionData, status) | `useContractDetail` → `ContractDetailData` | Phase 2a |

---

## ⚠️ Scope refinement (2026-07-03, v2 код тулгасны дараа)

v2 `contract-action-buttons` + parent handler-уудыг тулгахад:
- **review / approve** нь `VerificationModal("SUBMIT")` (2FA) + `enterFieldSideForm` (талбар бөглөх/гарын үсэг) дамждаг → **signing урсгал = 2d**, 2c биш.
- **accept-cancellation / delete / resend** нь `VerificationModal` (2FA) дамждаг → **2d dependency**.
- **return / cancel-request / decline-cancellation** нь зөвхөн `FeedbackModal` (reason текст) → **цэвэр POST, 2c-д хийнэ**.

**Тиймээс бодит 2c:**
1. `ActionButtons.tsx` — бүх товч + enable-логик (presentational, бүрэн).
2. `FeedbackModal` (reason input) + 3 mutation: return / cancel / decline.
3. Бусад товч (2FA/sign/pay) → placeholder handler prop → 2d/payment-д бодит болно.

**Exact payload (v2-оос):**
```
return:  POST /contract-action/return                { contractActionId, contractRequestId, reason }
cancel:  POST /contract-action/send-cancel-request   { contractRequestId, contractActionId, reason }
decline: POST /contract-action/decline-cancellation  { contractRequestId, contractActionId, reason }
// 2d (2FA): accept-cancellation {contractRequestId, contractActionId}, delete, resend, review/approve(+field form)
```
`contractActionId` = одоогийн `actionData.id` (route-ийн `actionId`); `contractRequestId` = `detail.id`.

---

## Ажлын зүйлс (Phase 2c)

### 1. Action API — `documents/api/client.ts`
Дараах POST endpoint-уудыг нэмэх (v3 prefix backend-тэй тулгах; v2 base = `BACKEND_URL` = v1):

```
returnContract(data)            → POST /public-api/v1/contract-action/return
resendContract(data)            → POST /public-api/v1/contract-request/resend
reviewContract(data)            → POST /public-api/v1/contract-action/review
approveContract(data)           → POST /public-api/v1/contract-action/approve
cancelContract(data)            → POST /public-api/v1/contract-action/cancel        (path backend-тэй тулга)
acceptCancellation(data)        → POST /public-api/v1/contract-action/accept-cancellation
declineCancellation(data)       → POST /public-api/v1/contract-action/decline-cancellation
// delete аль хэдийн бий: documentsApi.deleteContract (2a)
// participant resend аль хэдийн 2b-д (ParticipantAccordion) хойшлуулсан — resendNotification
```

> **Verify:** payload бүтэц (actionId / contractRequestId / reason) v2-оос тулга. Бүгд POST тул BFF okNoop асуудалгүй ([[E-Geree-v3-Networking-BFF]]).

### 2. Mutation hook-ууд — `documents/api/queries.ts`
`useReturnContract`, `useResendContract`, `useReviewContract`, `useApproveContract`, `useCancelContract`, `useAcceptCancellation`, `useDeclineCancellation`. Бүгд onSuccess → `queryClient.invalidateQueries(documentsKeys.detail(id, actionId))` (+ шаардлагатай бол receivedList).

### 3. `ActionButtons` компонент — `documents/components/detail/ActionButtons.tsx` (шинэ, client)
- Props: `detail`, `actionData` (одоогийн оролцогчийн action), handler-ууд.
- Enable-логик (v2 порт):

| Товч | Идэвхжих нөхцөл (v2) | 2c үйлдэл |
|---|---|---|
| Гарын үсэг зурах | `permissionType==="SIGN"` && status SIGN_PENDING | `onSign()` **placeholder → 2d** |
| Тоон гарын үсэг | status DIGITAL_SIGNATURE_PENDING | `onDigitalSign()` **placeholder → 2d** |
| Хянах (review) | permissionType review && REVIEW_PENDING | `useReviewContract` |
| Зөвшөөрөх (approve) | review/approve эрх | `useApproveContract` |
| Буцаах (return) | status ∈ [SIGN_PENDING, REVIEW_PENDING, ...] | `useReturnContract` + reason modal |
| Дахин илгээх (resend) | sender && PENDING төлөв | `useResendContract` |
| Цуцлах хүсэлт (cancel) | sender && идэвхтэй гэрээ | `useCancelContract` + confirm |
| Цуцлалт зөвшөөрөх/татгалзах | actionStatus==="CANCEL_PENDING" | `useAcceptCancellation` / `useDeclineCancellation` |
| Устгах (delete) | sender && устгаж болох төлөв | `deleteContract` (2a) + `DeleteConfirmDialog` |
| Төлөх (pay) | payer && PENDING төлбөр | `onPay()` **placeholder → payment** |

### 4. Detail layout-д холбох — `ContractDetail.tsx` / `ContractSidebar.tsx`
`ActionButtons`-г detail хуудсанд байрлуулах (footer эсвэл sidebar дээд). `actionData` = `actionList.find(a => a.id === actionId)` эсвэл related participant.

### 5. Reason / confirm modal
- Буцаах шалтгаан оруулах жижиг modal (эсвэл textarea + Button).
- Delete/cancel → `DeleteConfirmDialog` дахин ашиглах.

---

## Endpoint лавлагаа (v2-ээс)

| Үйлдэл | Method | Path (v2, BACKEND_URL=v1) |
|---|---|---|
| return | POST | `/contract-action/return` |
| resend | POST | `/contract-request/resend` |
| review | POST | `/contract-action/review` |
| approve | POST | `/contract-action/approve` |
| accept-cancellation | POST | `/contract-action/accept-cancellation` |
| decline-cancellation | POST | `/contract-action/decline-cancellation` |
| cancel (хүсэлт) | POST | `/contract-action/cancel` эсвэл `/contract-request/cancel` — **тулга** |
| delete | POST | `/contract-request/remove/{id}` (2a-д бий) |
| edit-resend | POST | `/contract-request/edit-resend` (edit урсгал — сонголт) |

---

## Verification (Phase 2c)

1. `npm run lint && npx tsc --noEmit` — цэвэр.
2. Товч enable/disable: янз бүрийн status/permission-той гэрээн дээр зөв товч гарах.
3. Return/resend/review/approve/cancel/accept-cancel/decline-cancel → API дуудаж, detail refetch, төлөв шинэчлэгдэх.
4. Delete → confirm → жагсаалт руу буцах.
5. Sign/digital/pay товч → placeholder (2d/payment гэсэн тэмдэглэлтэй).
6. `npm run knip` (phase төгсгөлд).
7. `npm run test:e2e` — critical-flows эвдрээгүй.

---

## Файлын товч жагсаалт

**Шинэ:**
- `src/features/documents/components/detail/ActionButtons.tsx`
- (сонголт) reason/return modal

**Засах:**
- `src/features/documents/api/client.ts` (action endpoints)
- `src/features/documents/api/queries.ts` (mutation hooks)
- `src/features/documents/components/ContractDetail.tsx` эсвэл `ContractSidebar.tsx` (ActionButtons холбох)

---

## v2 эх сурвалж (порт хийхэд)

- ActionButtons UI + enable-логик: `e-geree-v2/.../pages/contract-detail/contract-action-buttons/index.jsx` (242 мөр)
- Action API: `e-geree-v2/src/lib/actions/contract.js` (return L80, resend L85, approve L92, review L105, accept/decline-cancellation L123-131), `contractV2.js` (resendNotification)
- Public хувилбар: `e-geree-v2/.../public-contract/contract-detail/action-buttons`

---

## Гүйцэтгэлийн дараалал, модел

| # | Алхам | Файл | Effort | Модел |
|---|---|---|---|---|
| 2c-1 | Action endpoints | `api/client.ts` | medium | Sonnet 5 |
| 2c-2 | Mutation hooks | `api/queries.ts` | medium | Sonnet 5 |
| 2c-3 | `ActionButtons` + enable-логик | `detail/ActionButtons.tsx` | **high** | **Opus 4.8** |
| 2c-4 | Detail-д холбох + confirm/reason modal | `ContractDetail.tsx` | medium | Sonnet 5 |
| ✅ | verify + commit | — | — | — |

**Дүрэм:** 2c бүрэн (verify+commit) → 2d. Sign/digital/pay нь 2c-д зөвхөн placeholder — 2d/payment-д бодит болно. caveman + ponytail хэвээр.

### Чат стратеги
- **2c — ШИНЭ чат.** Seed: энэ note + `contract-action-buttons/index.jsx` + v3 `queries.ts`/`client.ts`.
- graphify эхэлж → зорилтот файл л унших.

---

## ✅ Гүйцэтгэсэн (2026-07-03, commit `7d4afe2`)

Subagent-аар model сольж гүйцэтгэсэн (cost-efficient):
- **Wave 1 (Sonnet):** `client.ts` 3 endpoint (return/send-cancel-request/decline-cancellation) + `queries.ts` 3 hook (`useReturnContract`/`useCancelContract`/`useDeclineCancellation`, detail invalidate).
- **Wave 2 (Opus):** `ActionButtons.tsx` (бүх товч + enable-логик v2-оос порт) + `FeedbackModal.tsx` (reason dialog, shadcn Dialog+Textarea).
- **Wave 3 (Sonnet):** `ContractDetail.tsx`-д ActionButtons холбосон (grid дээр bar; `actionData = actionList.find(id===actionId)`).

**Ажилладаг (2c):** буцаах / гэрээ цуцлах / хүсэлт татгалзах — FeedbackModal + mutation.
**Placeholder (2d/payment):** sign/review/approve/digital-sign/delete/resend/edit/accept-cancellation → no-op prop; pay → payment. `taskVerified=true` (DAN/participantsConfig 2d-д).

**Verify:** tsc 0, eslint цэвэр, knip цэвэр.

**⚠️ Backend тулгах өр:** endpoint 3-ын payload/prefix; `send-cancel-request` path; accessCode encrypt (2a).
