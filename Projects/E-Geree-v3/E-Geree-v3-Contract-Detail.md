---
title: "Contract Detail — гэрээний дэлгэрэнгүй хуудас"
type: project
status: draft
created: 2026-07-07
updated: 2026-07-07
tags:
  - project
  - project/e-geree-v3
---

# Contract Detail — гэрээний дэлгэрэнгүй хуудас

[[E-Geree-v3-Home]]

`/documents/detail/...` хуудасны тогтвортой бүтэц (Phase 2a → 2d, 2026-07-02…07-07-нд баригдсан). Энэ нь permanent reference — идэвхтэй ажлын явц нь [[E-Geree-v3-Contract-Detail-Page-Phase2c-Plan]] ба [[E-Geree-v3-Contract-Detail-Page-Phase2d-Plan]]-д.

## Route бүтэц — `[...slug]` triple parse

`src/app/[locale]/(protected)/(with-sidebar)/documents/detail/[...slug]/page.tsx` — server component, slug-ийг гурав задалж client компонент руу дамжуулна (L10):

```tsx
const [contractId, actionId, accessCode] = slug ?? [];
→ <ContractDetail contractId=… actionId=… accessCode=… />
```

- v2-той ижил хэлбэр: `/documents/detail/{contractId}/{actionId}[/{accessCode}]`.
- Navigation эх сурвалж: received жагсаалтын drawer ("Дэлгэрэнгүй харах" / rep-modal сонголт) — [[E-Geree-v3-Contract-Detail-Drawer-Phase1-Plan]].

## Data fetch — `getContractDetail` + React Query

Documents feature бүхэлдээ **client-side React Query** (v2-ийн server-action-аас ялгаатай, v3 pattern):

- `documentsApi.getContractDetail` — `src/features/documents/api/client.ts:29-32`:
  `GET /public-api/v1/contract-request?contractRequestId={id}&actionId={actionId}[&accessCode=…]` (`apiClient`, `baseURL:"/backend"` — [[E-Geree-v3-Networking-BFF]]).
- `useContractDetail(contractId, actionId, accessCode?)` — `src/features/documents/api/queries.ts:31-40`. Key = `documentsKeys.detail(contractId, actionId)` (`["contract-detail", id, actionId]`, queries.ts:20-21) + `accessCode` key-д орсон тул код өөрчлөгдөхөд автоматаар дахин татна. `enabled: Boolean(contractId && actionId)`.
- Envelope дүрэм: `apiClient` interceptor хариуг unwrap хийдэг тул v2-ийн `data.X` хандалт v3-т шууд `.X` болно (`coreFetcher` нь v2 `httpRequest`-тэй ижил `{isOk, data}` барьдаг).
- Нэмэлт файл: `getAdditionalFileList` (client.ts:36-39, `GET /public-api/v1/contract-addition-file/list?contractId=…&type=SENT`) + `useAdditionalFiles` (queries.ts:43-48).

## Gate-ууд (`ContractDetail.tsx:143-146`)

Дараалал (client-т, v2 server gate-ийн буулгалт):

1. `!contractId || !actionId` → `PermissionDenied` (түгжээ icon + "Жагсаалт руу буцах").
2. `isLoading` → `DetailSkeleton` (2 баганат skeleton).
3. `isError || !data` → `PermissionDenied`.
4. `data.secure` → `SecuredDetailContent` (accessCode gate).

**Secured gate механизм** (`src/features/documents/components/SecuredDetailContent.tsx`): код оруулаад `router.replace(`${pathname}/${code}`)` — URL-д хавсаргаж route slug дахин parse → `useContractDetail` accessCode-той refetch.

> **Мэдэгдэж буй өр (тогтвортой зан):** буруу accessCode үед backend дахин `secure:true` буцааж gate **чимээгүй** дахин гарна — алдааны toast/мессеж алга (SecuredDetailContent.tsx:10-13 comment). Мөн v2-ийн `encryptPassword`-ийг v3 портлоогүй, accessCode raw дамжина (client.ts:28 ponytail тэмдэглэл) — backend compat батлагдаагүй таамаг.

## Layout — 2 багана + нэгдсэн dock

`src/features/documents/components/ContractDetail.tsx`. Bounded flex (`lg:h-[calc(100svh-5rem)]`), нэг bordered карт дотор:

- **Toolbar** (h-14): identity only — `data.title`. Үйлдэл энд БАЙХГҮЙ.
- **Grid** `lg:grid-cols-[1fr_380px]` (L279): зүүн **PDF hero** — `ViewerPlaceholder` → `PdfViewer` (view-only + live `fields`; `pdfViewUrl = signedPdfUrl || generatedPdfUrl || relatedPdfUrl`, `resolvePdfUrl` — ViewerPlaceholder.tsx:15,29; [[E-Geree-v3-PDF-Viewer]]); баруун **контекст rail** (тусдаа scroll).
- **Rail footer**: `ActionButtons` sticky dock.

## Rail tabs — Бөглөх | Мэдээлэл

`fillActive` (= `isFieldFillActive(detail, actionData)`) үед rail нь controlled `Tabs` (`railTab` state, ContractDetail.tsx:284-311):

- **Бөглөх** (default) — `FieldFillPanel` (`detail/FieldFillPanel.tsx`): `shared/field-fill`-ийн `SenderFillSection`-г wrap хийж өөрийн `participantKey`-ийн `FILLABLE_TYPES` талбарыг л үзүүлнэ. Гарын үсэг = `auth.user.signatureImgUrl` preset apply (зурах UI хэрэггүй). Sticky header (гарчиг+заавар scroll-д алдагдахгүй).
- **Мэдээлэл** — `ContractSidebar` (оролцогч / нэмэлт файл / түүх; оролцогчийн accordion нь drawer-тэй дундын `ParticipantAccordion.tsx`).
- `!fillActive` үед tab-гүй, `ContractSidebar` шууд (нэг зорилго → tab chrome илүүц).
- **Fill-guard:** талбар дутуу (`fieldsIncomplete`) байхад Баталгаажуулах дарвал сүлжээ хүсэлт гаргалгүй `setRailTab("fill")` (L194-196).
- Талбарын утга **parent-ийн local `fields` state**-д (tab unmount-д алдагдахгүй); server refetch бүрд render-time compare-and-set-ээр дахин seed (L92-99, `seededFieldList` idiom).

## Нэгдсэн action dock — `ActionButtons`

`src/features/documents/components/detail/ActionButtons.tsx`. Түүх: layout redesign (`9e6409b`) `FlowActionDock` тусдаа авчирсан ч commit `c2c07e5`-д **устгаж `ActionButtons`-д нэгтгэсэн** — одоо нэг л dock `[primary (flex-1)][⋯ overflow]` rail footer-т (sticky, L296).

**Primary resolver — яг нэг primary** (L239-255):
1. `getFlowAction(detail, actionData)` → Хянах / Баталгаажуулах / Тоон гарын үсэг;
2. байхгүй бол fallback: Дахин илгээх → Хүсэлт зөвшөөрөх → Төлбөр төлөх;
3. юу ч алга бол primary-гүй, зөвхөн "Үйлдэл" overflow товч.

**`⋯` overflow** (L259-290): Буцаах · Дахин илгээх* · Засах · Хүсэлт зөвшөөрөх* · Хүсэлт татгалзах (destructive) · Төлбөр төлөх* · Гэрээ цуцлах (destructive) · Устгах (destructive). (* = primary болоогүй үед л.)

### Flow үйлдлийн шийдэгч — `lib/action-status.ts`

`src/features/documents/lib/action-status.ts`:

- `isFieldFillActive` (L5-18): `detail.status ∈ {PENDING, RESENT_PENDING, EDITED_PENDING}` && `actionData.status === "SIGN_PENDING"`.
- `getFlowAction` (L27-48): `REVIEW_PENDING` → review; `SIGN_PENDING` → approve; `detail.status ∈ {DIGITAL_SIGNATURE_PENDING, DIGITAL_SIGNED}` && `actionStatus === "DIGITAL_SIGNATURE_PENDING"` → digitalSign. Харилцан үл давхцана — нэг л буцаана.
- `contractStatusMeta` (L51-66): статус → шошго + badge өнгөний map.

### Товчны enable-логик (v2 порт, ActionButtons.tsx:103-136)

| Товч | Идэвхжих нөхцөл |
|---|---|
| Буцаах (return) | `status ∈ {PENDING, RESENT_PENDING, EDITED_PENDING, SIGNED}` |
| Дахин илгээх (resend) | `status === RETURNED` && `actionData.sender` |
| Устгах (delete) | `status ∈ {PENDING, RESENT_PENDING, EDITED_PENDING, SIGNED, RETURNED, DIGITAL_SIGNATURE_PENDING, DIGITAL_SIGNED}` && `sender` |
| Гэрээ цуцлах (cancel) | `status === COMPLETED` |
| Хүсэлт зөвшөөрөх/татгалзах | `actionStatus === "CANCEL_PENDING"` |
| Төлбөр төлөх (pay) | `actionData.payer` && `paymentType === "PAY"` && `paymentStatus === "PENDING"` && flow action алга |

**Бизнес дүрэм:** client-side sign-gate v3-т ЗОРИУДААР алга — v2-ийн `taskVerified` (DAN) нь v3-т dead type талбар (`auth-types.ts`), баталгаажуулалтыг backend `methods`-оор enforce хийдэг (2026-07-07 audit-аар батлагдсан).

## Reason-based үйлдлүүд (Phase 2c)

3 үйлдэл нь 2FA-гүй, зөвхөн шалтгаан (`FeedbackModal` — `detail/FeedbackModal.tsx`, Dialog+Textarea) шаарддаг цэвэр POST. Payload `{contractActionId, contractRequestId, reason}` (`contractActionId` = route-ийн `actionId` = `actionData.id`; `contractRequestId` = `detail.id`):

| Үйлдэл | Endpoint (client.ts) | Hook (queries.ts) |
|---|---|---|
| return | `POST /public-api/v1/contract-action/return` (L67-68) | `useReturnContract` (L114) |
| cancel-request | `POST /public-api/v1/contract-action/send-cancel-request` (L70) | `useCancelContract` (L127) |
| decline-cancellation | `POST /public-api/v1/contract-action/decline-cancellation` (L71-72) | `useDeclineCancellation` (L140) |

Бүх mutation onSuccess → `documentsKeys.detail` invalidate.

**Confirm-based үйлдлүүд** (2d-1; v2-т эдгээр OTP биш зөвхөн confirm modal байсан) — `ConfirmActionDialog` (generic AlertDialog):

- delete: `POST /public-api/v2/contract-request/remove/{id}` (client.ts:24-25) → амжилтад `/documents/received` руу.
- resend: `POST /public-api/v1/contract-request/resend` `{contractRequestId}` (client.ts:75).
- accept-cancellation: `POST /public-api/v1/contract-action/accept-cancellation` `{contractRequestId, contractActionId}` (client.ts:73-74).

## 2FA/OTP submit урсгал (Phase 2d-2c)

Хоёр үе шаттай (login 2FA-ийн `verify:true` pattern-тай ижил), `ContractDetail.tsx:164-241` orchestrate хийнэ:

1. `verify:false` payload → `POST /public-api/v1/contract-action/approve|review` (client.ts:57-64).
   - approve payload: `{verify, codes?, contractActionId, participantKey, contractRequestId, contractFieldList}` (бөглөсөн `fields` дагалдана);
   - review payload: `{verify, codes?, contractActionId, contractRequestId}`.
2. Хариу `methods` (ж: `{EMAIL:true,"2FA":true}`) агуулбал → `SecurityVerificationModal` (`detail/SecurityVerificationModal.tsx` — dynamic multi-method OTP, shadcn `input-otp`, countdown `date-fns`, retry) нээгдэнэ.
3. `verify:true` + `codes:{…}` → эцсийн амжилт (SIGNED) → toast + detail invalidate.

- `useApproveContract`/`useReviewContract` (queries.ts:55-82) — `!data.methods` (эцсийн амжилт) үед л invalidate.
- **Бизнес дүрэм:** `methods` нь хэрэглэгчийн **идэвхжүүлсэн verification арга** (EMAIL/SMS/2FA)-аар gate хийгдэнэ, `danVerified`-тай ХАМААГҮЙ; арга идэвхгүй хэрэглэгчид `verify:false` шууд SIGNED болгодог. 2026-07-07-нд live-баталгаажсан.

## Тоон гарын үсэг (Phase 2d-3 GSIGN / 2d-3b TRIDUM)

Upstream = v2-тэй ижил хост: `digital-signature-api.e-geree.mn/digital-signature-api/v1`.

- Config: `getDigitalSignatureUrl()` — `src/core/config/index.ts:52` (`DIGITAL_SIGNATURE_URL` env, base version-baked).
- BFF proxy: `src/app/backend/digital-signature/[...path]/route.ts` (POST+GET).
- API (client.ts): `createSignRequest` (L81, `POST /pdf-sign-request/create`, `type: "GSIGN"|"TRIDUM"`) · `sendGSignRequest` (L86, `GET /g-sign/contract/sign?phoneNumber=&pdfSignRequestId=`) · `getSignedUrlByOtp` (L93, `GET /signed-pdf/url-by-otp`) · `registerDigitalSignCert` (L95, `POST /signed-pdf/save-serial-number`).
- `useStartGSign` (queries.ts:90-111): create + g-sign push **нэг мутаци**, буцаана = socket room key (`pdfSignRequestId`); `id/otp/inputPdfUrl` guard; invalidate ХИЙХГҮЙ — эцсийн баталгаа socket-оор.
- **GSIGN modal** — `detail/DigitalSignModal.tsx`: утас `user.userProfile.mobile`-аас preset → DAN push → socket хүлээлт. Socket контракт (`socket.io-client@^4`): `io(`${NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL}/digital-signature`, {transports:['websocket'], withCredentials:true})` → `connect` дээр `emit('join-room',{room:pdfSignRequestId})` (L95) → `on('sign', JSON.parse → {result,message})` (L97); `"User cancelled the request"` = татгалзсан. v2-ийн 2 bug зассан: socket leak → useEffect cleanup `disconnect()`; transport failure hang → `connect_error` handler.
- **TRIDUM** — `detail/use-tridum-sign.ts`: суусан desktop клиент рүү **шууд** (BFF-гүй) raw WebSocket `ws://127.0.0.1:59001` (L6). Урсгал: version probe `{Command:'99'}` (L73) → `{Command:'4', OTP, SignLocation, PDFFiles}` (L105) → амжилтад `getSignedUrlByOtp` + `registerDigitalSignCert`. Гарын үсгийн байрлал = `lib/tridum.ts` `calcTridumLocation(actionList)` — `DIGITAL_SIGNED` action-ы тоогоор 2 баганад шатлана.
- `ContractDetail.tsx`: `handleDigitalSigned` (L245-253) — socket амжилтад detail invalidate + toast.

## Төлбөрийн урсгал (Phase 2d-4)

v2-ийн глобал `PaymentProvider` context-ийн оронд **self-contained modal** (нэг хэрэглэгчтэй тул).

- Config: `getPaymentUrlV2()` — `src/core/config/index.ts:42` (`payment/initiate` зөвхөн /v2-т; `PAYMENT_URL_V2` env эсвэл `PAYMENT_URL`-ийн /v1→/v2 regex fallback).
- BFF proxy: `src/app/backend/payment-v2/[...path]/route.ts` (POST+GET).
- API (client.ts): `getPaymentByContractRequest` (L99) · `getPaymentMethodList` (L101) · `paymentInitiate` (L102, /payment-v2 route-оор) · `checkPaymentStatus` (L104).
- **Modal** — `detail/ContractPaymentModal.tsx`: дүн + аргын жагсаалт (useQuery) → initiate → **QPay** бол QR(base64)+deeplink + payment socket (`${base}/payment`, room=`number`, `pending`→success); **redirect** арга (SOCIAL_PAY/BANK_CARD/ARD) бол шинэ таб; "Төлбөр шалгах" = `checkPaymentStatus`. НӨАТ баримтын төрөл оролцогчийн профайлаас авто.
- `ContractDetail.tsx` `handlePaid` (L257-265) — invalidate + toast.

## Breadcrumb — shared context slot

`Баримт бичиг → Ирсэн/Илгээсэн → [гэрээний нэр] → Оролцогч N`

- **Boundary шалтгаан:** `DynamicBreadcrumb` нь `src/components/layout/dynamic-breadcrumb.tsx` (app-component) — feature hook импортлож болохгүй. Тиймээс дундын slot: `src/shared/breadcrumb/` (`breadcrumb-context.tsx` + `index.ts`) — `BreadcrumbProvider` / `useBreadcrumb` / `useSetBreadcrumb`.
- Provider: `(with-sidebar)/layout.tsx:18` wrap. Consumer: `dynamic-breadcrumb.tsx:33` — context items байвал custom, эс бол pathname fallback.
- `useSetBreadcrumb` семантик (breadcrumb-context.tsx:51-58): layout effect (paint-ээс өмнө, fallback анивчихгүй); unmount-д `null` цэвэрлэнэ; `[]` = breadcrumb бүр нуух (data ачаалж дуустал); items-ийг дуудагч `useMemo`-оор тогтворжуулна.
- `ContractDetail.tsx:123-141`: `data.title` + `actionData.sender` (sender → "Илгээсэн" `/documents/sent`, эс бол "Ирсэн" `/documents/received`) + `getParticipantNumber(participantKey)`.

## Файлын зураглал

| Үүрэг | Зам |
|---|---|
| Route | `src/app/[locale]/(protected)/(with-sidebar)/documents/detail/[...slug]/page.tsx` |
| Orchestrator | `src/features/documents/components/ContractDetail.tsx` |
| Secured gate | `src/features/documents/components/SecuredDetailContent.tsx` |
| PDF талбай | `src/features/documents/components/detail/ViewerPlaceholder.tsx` |
| Rail (мэдээлэл) | `detail/ContractSidebar.tsx`, `ParticipantAccordion.tsx` |
| Rail (бөглөх) | `detail/FieldFillPanel.tsx` + `src/shared/field-fill/SenderFillSection.tsx` |
| Action dock | `detail/ActionButtons.tsx` + `lib/action-status.ts` |
| Modals | `detail/FeedbackModal.tsx`, `ConfirmActionDialog.tsx`, `SecurityVerificationModal.tsx`, `DigitalSignModal.tsx`, `ContractPaymentModal.tsx` |
| TRIDUM | `detail/use-tridum-sign.ts`, `lib/tridum.ts` |
| API | `src/features/documents/api/client.ts`, `queries.ts` |
| Breadcrumb | `src/shared/breadcrumb/`, `src/components/layout/dynamic-breadcrumb.tsx` |
| BFF | `src/app/backend/digital-signature/[...path]/route.ts`, `src/app/backend/payment-v2/[...path]/route.ts` |

**Env (deploy шаардлага):** `DIGITAL_SIGNATURE_URL`, `NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL`, `PAYMENT_URL_V2` (сүүлийнх нь optional — /v1→/v2 regex fallback бий). Локал `.env.production`/`.env.development`-д бий, gitignored.

## Холбоо

- Идэвхтэй plan-ууд: [[E-Geree-v3-Contract-Detail-Page-Phase2c-Plan]] · [[E-Geree-v3-Contract-Detail-Page-Phase2d-Plan]]
- [[E-Geree-v3-PDF-Viewer]] — viewer + field-fill дундын модулиуд
- [[E-Geree-v3-Networking-BFF]] — apiClient envelope, BFF proxy-ууд
- [[E-Geree-v3-Routing]] · [[E-Geree-v3-State-Management]] · [[E-Geree-v3-Contract-Detail-Drawer-Phase1-Plan]]
