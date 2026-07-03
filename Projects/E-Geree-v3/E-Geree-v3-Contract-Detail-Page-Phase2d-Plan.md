---
title: "Plan: Contract Detail Page — Phase 2d (signing + 2FA + digital signature)"
type: project
status: draft
created: 2026-07-03
updated: 2026-07-03
tags:
  - project
  - project/e-geree-v3
  - plan
---

# Plan: Contract Detail Page — Phase 2d (гарын үсэг + 2FA + тоон гарын үсэг)

**Огноо:** 2026-07-03
**Салбар:** dev-khishigee
**Төлөв:** 📝 Ноорог (батлагдаагүй)
**Холбоотой:** [[E-Geree-v3-Contract-Detail-Page-Phase2a-Plan]] · [[E-Geree-v3-Contract-Detail-Page-Phase2c-Plan]] · [[pdf-viewer-new]] · [[E-Geree-v3-Networking-BFF]]

---

## Context (яагаад)

2a (route+gate) ✅ · 2b (sidebar accordion) ✅ `7a533e7` · 2c (ActionButtons — reason actions) ✅ `7d4afe2`. 2c-д **placeholder no-op** үлдсэн товчнууд: sign / review / approve / digital-sign / delete / resend / edit / accept-cancellation / pay.

Phase 2d = эдгээрийн **2FA/гарын үсгийн урсгалыг** гүйцээх. Энэ бол хамгийн том, эрсдэлтэй фаза — **олон sub-phase болгон задална**, тус бүр ШИНЭ чат (магадгүй тусдаа note).

### 2d-ийн task-ууд (v2-оос)
- **2FA verification modal** (OTP/баталгаажуулах) — `action-verification-modal` + `sercurity-verification-modal`. delete/resend/accept-cancellation/submit-ийг хамгаална.
- **Sign flow** — талбар бөглөх (`contractFieldList` + `enterFieldSideForm`) → OTP verify (`sendVerificationRequest`) → `successSend`. sign/review/approve.
- **signature-input** — гарын үсэг зурах/сонгох (хадгалсан гарын үсэг).
- **Digital signature** — `DigitalSignature`, g-sign (утас + socket + e-mongolia/DAN), `createSignRequest`/`sendRequestGSign`.
- **DAN verification** — `taskVerified` (одоо 2c-д hardcoded `true`).

---

## ⚠️ Гол хамаарал / хориг (ЗААВАЛ эхэнд шийдэх)

| Хамаарал | Юунд хэрэгтэй | Төлөв |
|---|---|---|
| **[[E-Geree-v3-PDF-Viewer\|pdf-viewer]] (view-only wiring)** | Detail хуудсанд PDF харуулах | ✅ DUUSSAN (2026-07-03, commit `e928bc5`) — `ViewerPlaceholder` → `PdfViewer`, `resolvePdfUrl` shared болсон |
| **pdf-viewer талбар (fields prop) ашиглах** — `isEdit`/`edit` config дээр interactive fill | Sign flow-д `contractFieldList` бөглөхөд PDF талбарын drag/select UI хэрэгтэй | ⛔ 2d-2b-ийн хориг (view-only дууссан ч edit/drag хэсэг шинэ) |
| `ContractDetailData.config.participantsConfig`, `contractFieldList` | taskVerified (DAN), sign payload, талбар display/fill | Type өргөтгөх шаардлагатай (2a-д хойшлуулсан) |
| Хадгалсан гарын үсэг (profile signature) | signature-input сонголт | v2 `profile/user/signature` — v3-т байгаа эсэх тулга |
| DAN verified flag (`authUser.danVerified`) | taskVerified | v3 auth/profile-д байгаа эсэх тулга |

**Дүгнэлт:** **2d-1 (verification modal)** ✅ дууссан. **pdf-viewer view-only wiring** ✅ дууссан → **2d-2a (талбар ДИСПЛЕЙ)** одоо нээлттэй, шууд эхэлж болно. **2d-2b (талбар ФИЛЛ/edit)**, **2d-2c (OTP submit)**, **2d-3 (digital)** дараалан хийнэ.

---

## Sub-phase задаргаа

### ✅ 2d-1: DUUSSAN (2026-07-03, commit `53f43b4`)
**Нээлт:** v2-т delete/resend/accept-cancellation нь **confirm modal** (OTP БИШ) — `ActionVerificationModal`. OTP (`SecurityVerificationModal`) нь зөвхөн SIGN/SUBMIT урсгалд (2d-2). Тиймээс 2d-1 = confirm dialog.
- `client.ts`: `acceptCancellation` `{contractRequestId, contractActionId}`, `resendContract` `{contractRequestId}`.
- `queries.ts`: `useAcceptCancellation`, `useResendContract` (detail/received-list invalidate).
- **New** `ConfirmActionDialog.tsx` (generic AlertDialog confirm).
- `ActionButtons.tsx`: delete/resend/accept-cancellation → confirm dialog + mutation дотооддоо. delete амжилтад `/documents/received` руу. onDelete/onResend/onAcceptCancellation prop устгасан.
- Subagent: 2 wave Sonnet (endpoints/hooks + dialog/wiring) — Opus хэрэггүй (pattern-following).
- Verify: tsc 0, eslint цэвэр, knip цэвэр.

**Placeholder хэвээр (2d-2/2d-3/payment):** sign/review/approve/digital-sign/edit/pay.

### 🟢 (лавлагаа) 2d-1 анхны төлөвлөлт — Verification modal (2FA)
- `ActionVerificationModal` порт (confirm + OTP оруулах). v2: `components/custom/modals/action-verification-modal`, `pages/auth/sercurity-verification-modal`.
- 2c placeholder-уудыг холбох: **delete** (`useDeleteContract` аль хэдийн бий), **resend**, **accept-cancellation** (`acceptCancellation` endpoint нэмэх).
- Урсгал: товч → verification modal → OTP/баталгаажуулалт → mutation → detail invalidate.
- **Үр дүн:** delete/resend/accept-cancellation ажиллана. Effort: **high**.

### 🟡 2d-2: Гэрээний талбаруудын логик (fields) — 3 дэд алхамд задарсан

**Нээлт (2026-07-03):** `PdfViewer` нь `fields`/`hideFields`/`fieldStyle` prop-оор **view-only талбар display**-г аль хэдийн дэмждэг (`PdfPage.tsx` → `isEdit=false` үед `PdfViewerPage` рүү дамжина, drag/edit биш зөвхөн зурна). Interactive fill (чирэх/сонгох) л `isEdit`+`edit` (`PdfEditConfig`) шаардана. Тиймээс **display ба fill хоёр өөр ажлын хэмжээтэй** — тусад нь sub-phase болгов.

#### ✅ 2d-2a: Талбар ДИСПЛЕЙ (read-only) — DUUSSAN (2026-07-03, commit `43db7df`)
- Type өргөтгөл: `ContractDetailData.contractFieldList?: ContractField[]` — **contract-create-ийн биш**, `shared/pdf-viewer`-ийн `ContractField` (type-layering: global types feature дотоод рүү импортлохгүй; pdf-viewer нь аль хэдийн 2 feature-д нийтлэг).
- `ViewerPlaceholder.tsx`: `<PdfViewer pdfUrl={...} fields={detail.contractFieldList} fieldStyle={...} />` — `isEdit`/`edit` дамжуулаагүй тул автоматаар read-only зурагдана.
- `fieldStyle`: `getParticipantColor`, `SubmitStep.tsx`-тэй ижил pattern.
- **Нээлт:** `getParticipantColor` эхлээд `contract-create/lib`-д байсан тул `boundaries/element-types` eslint дүрэм зөрчсөн (feature бусад feature-ийн дотоод импортлохгүй). Шийдэл: `src/lib/participant-colors.ts` руу зөөж (7 callsite шинэчилсэн) — `pdf-url.ts`-тэй ижил хэв маяг (cross-feature util → shared/root lib).
- Subagent: хэрэггүй (жижиг, шууд хийсэн). Verify: tsc 0, eslint цэвэр (boundaries орсон), knip шинэ мэдэгдэлгүй.
- **Үр дүн:** гэрээн дээрх бөглөсөн/хоосон талбарууд PDF дээр харагдана (ямар ч edit-гүй).

#### ✅ 2d-2b: Талбар ФИЛЛ — DUUSSAN (2026-07-03, commit `b4ece39`)
**Нээлт:** анхны төлөвлөгөө `isEdit`+`PdfEditConfig` (drag/select) шаардана гэж таамагласан нь БУРУУ байсан. `SubmitStep.tsx` (contract-create) аль хэдийн яг ижил асуудлыг (илгээгчийн ӨӨРИЙН талбарыг бөглөх, илгээхийн өмнө) шийдсэн байсан — **`SenderFillSection`**: sidebar ФОРМ (text/date/number/select/signature/stamp-зураг тус бүрийн input widget) + `PdfViewer`-г **view-only** (isEdit-гүй) орхиод `fields` prop-оор нь value-г live харуулна (`FieldsOverlay` `field.value`-г аль хэдийн зурдаг). Гарын үсэг ч зурах UI хэрэггүй — `auth.user.signatureImgUrl`-г шууд `value`-д "apply" хийдэг (preset зураг). Тиймээс 2d-2b = **PdfEditConfig/drag огт хэрэггүй**, зөвхөн pattern дахин ашиглалт.
- `ContractDetail.tsx`: local `fields` state (`data.contractFieldList` хуулбар, render-т харьцуулж дахин seed — `FieldsStep.tsx`-ийн `prevErrorPk` pattern, `useEffect`+`setState` биш eslint `react-hooks/set-state-in-effect`-д баригдсан тул).
- `ViewerPlaceholder.tsx`: `fields?: ContractField[]` prop нэмж, `detail.contractFieldList`-ийн оронд эдгээр local утгыг `PdfViewer`-т дамжуулна (live preview).
- **New** `FieldFillPanel.tsx` (documents/detail) — `SenderFillSection`-г wrap хийж, `actionData.participantKey`-гаар ӨӨРИЙН талбарыг filter хийнэ (`FILLABLE_TYPES`).
- **New** `features/documents/lib/action-status.ts`: `isFieldFillActive(detail, actionData)` — `ActionButtons`-ийн inline `activeFieldSide` тооцооллыг hoist хийж 2 газар (ActionButtons + FieldFillPanel) нэг эх сурвалжтай болгов.
- **Том нээлт (architecture):** `boundaries/element-types` eslint дүрэм нь **`index.ts`-ээр export хийсэн ч** feature→feature импортыг хориглодог (`src/features/*` бүхэлдээ нэг "feature" элемент — дэд зам харгалзахгүй). Тиймээс `SenderFillSection`-г бодитоор **contract-create-ээс гаргаж** `src/shared/field-fill/` (шинэ модуль, `pdf-viewer`-тэй ижил хэв маяг) руу зөөв; түүний дотоод хамаарал `fieldLabelKey` (`lib/labels.ts`) болон `uploadFieldImage` (`api/upload.ts`) мөн `src/lib/` руу зөвлөгдсөн (5+2 callsite шинэчилсэн, бүгд contract-create дотор хэвээр импортолно, зөвхөн эх сурвалжийн байршил өөрчлөгдсөн).
- `SenderFillSection`-д `description?: string` optional prop нэмсэн (contract-create-ийн `t("senderFillDesc")` = "илгээхээс өмнө..." гэсэн sender-specific текстийг sign-step-д ашиглахгүйн тулд; documents feature next-intl хэрэглэдэггүй тул raw Монгол текст дамжуулна).
- Subagent: хэрэггүй, шууд хийсэн (табан effort "Opus/high" гэж төлөвлөсөн ч бодит хэрэгжилт нь Sonnet-төвшний pattern-reuse болсон).
- Verify: tsc 0, eslint цэвэр (boundaries орсон), `npx vitest run` 28/28 (labels.test.ts зөөгдсөн ч тэнцсэн), knip шинэ мэдэгдэлгүй.
- **Үр дүн:** SIGN_PENDING оролцогч талбараа бөглөж чадна (сервер рүү илгээхгүй, зөвхөн local — 2d-2c нь илгээнэ), PDF preview live шинэчлэгдэнэ.

#### ✅ Layout redesign — DUUSSAN (2026-07-03, commit `9e6409b`)
2a–2d-2b-ийн дараа хуудсыг дизайны хувьд эмхэлсэн (product-design-preflight → батлагдсан plan `contract-detail-moonlit-walrus.md`). Хэрэглэгчээр батлагдсан: **2-col hero + контекст rail** ба **two-tier action**.
- **Layout:** PDF hero, bounded 2-col `lg:grid-cols-[1fr_380px]` (өмнөх full-width action-row + `[1fr_360px]`-ийн оронд). Баруун rail нэг: sign үед `FieldFillPanel` → meta (оролцогч/файл/түүх), тусдаа скролл. **3-col хийгээгүй** — wizard-ийн 3-col нь `(without-sidebar)`-д, detail нь `(with-sidebar)`-т тул PDF шахагдана.
- **Two-tier action:** flow товч (Approve/Review/Digital-sign) шинэ `FlowActionDock`-т rail-ийн ёроолд docked (бөглөх талбарын дараа шууд — fill→approve split зассан). Management товч (Буцаах/Дахин илгээх/Засах/Зөвшөөрөх + destructive Устгах/Цуцлах/Татгалзах overflow-д) дээд toolbar-т; toolbar route-д дутуу байсан title + status badge + copy-ID-г нөхнө.
- **Файлууд:** `lib/action-status.ts` (+`getFlowAction`, +`contractStatusMeta`), шинэ `FlowActionDock.tsx`, `ActionButtons.tsx` (document-actions only, flow prop хассан, destructive→`DropdownMenu`), `ContractDetail.tsx` (bounded flex, toolbar, rail, skeleton), `ViewerPlaceholder` (`h-[60vh] lg:h-full`), `ContractSidebar` (aside wrapper хассан).
- **Хамрахгүй:** бодит sign/OTP урсгал 2d-2c хэвээр — dock товч placeholder handler дуудна.
- Verify: tsc 0, eslint цэвэр (boundaries), knip шинэ флаггүй, vitest 28/28. Визуал (state бүр) manual `npm run dev`.

#### ✅ 2d-2c: OTP verify + submit finalize — DUUSSAN (2026-07-03, commit `8dd7531`)
**Нээлт:** v2-ийн `approveContract`/`reviewContract` (`{BACKEND_URL}/contract-action/approve|review`) нь **v3-тай ЯГ ИЖИЛ backend** (`public-api.e-geree.mn`) дээр ажилладаг production-д батлагдсан endpoint — "backend тулга" гэсэн эрсдэл бодит дээрээ байхгүй байсан (login 2FA-ийн `verify:true` two-phase pattern v3-т аль хэдийн байгаа — `login-form.tsx`). Тиймээс backend confirmation алхам алгасагдаж шууд ажил эхэлсэн.
- `client.ts`: `approveContract`/`reviewContract` нэмсэн (`ApproveContractPayload`/`ReviewContractPayload`/`ContractActionVerifyResult` type-ууд). Хариу v3-ийн `apiClient` interceptor-ээр аль хэдийн unwrap хийгдсэн тул `res?.result` шалгах шаардлагагүй (throw хийвэл reject, амжилттай бол шууд `data`).
- `queries.ts`: `useApproveContract`/`useReviewContract` — `onSuccess`-д `methods` алга бол (эцсийн амжилт) л detail invalidate.
- **New** `SecurityVerificationModal.tsx` — v2-ийн адил dynamic OTP (олон `methods` key дэмждэг), countdown (`date-fns` `differenceInSeconds`), `retrySendCode`. v3-ийн бодит shadcn `input-otp` (`InputOTPGroup`+`InputOTPSlot`, v2-ийн illat API-аас өөр) ашигласан.
- `ContractDetail.tsx`: хоёр үе шаттай урсгал (`buildApprovePayload`/`buildReviewPayload` → `verify:false` → `methods` ирвэл modal нээх → `verify:true+codes`). `errMsg` local helper (кодын house-style, project даяар давхардсан pattern).
- **i18n:** `security.*`/`actions.*` бүх шаардлагатай key (`verification`, `verificationDesc`, `retrySendCode`, `expireDate`, `emailVerificationCode` гэх мэт) v3 locale-д АЛЬ ХЭДИЙН байсан (v2-ээс бүтнээр хуулагдсан) — нэмэлт орчуулга хэрэггүй болсон.
- **Хамрахгүй:** `signature-input` (гарын үсэг зурах UI) — энэ бол 2d-3 (digital signature)-ийн хамрах хүрээ, 2d-2c-д шаардлагагүй (SenderFillSection аль хэдийн `signatureImgUrl`-г 2d-2b-д preset хийсэн). Supplement file upload (v2 `successSend`-ийн participant1 salaa) мөн хамрагдаагүй — тусдаа асуудал биш тул YAGNI.
- Verify: tsc 0, eslint 0 (шинэ файлууд цэвэр, 4 pre-existing warning өөр файлд), vitest 28/28.
- **Үр дүн:** sign/review OTP хүртэл бүрэн ажиллана. Гар аргаар (browser) шалгагдаагүй — authenticated session + SIGN_PENDING/REVIEW_PENDING бодит гэрээ шаардана.
- **Дараагийн алхам:** 2d-3 (digital signature, g-sign/DAN) — 2d-2c-аас хамааралтай, одоо нээлттэй.

### ⏸️ 2d-3: Digital signature (g-sign / DAN) — ХОЙШЛУУЛСАН (2026-07-03, хэрэглэгчийн шийдвэр)
**Хэрэглэгч:** "2d-3, 2d-4 эдгээрийг бүүр дараа хийнэ. Тэр болтол тэмдэглээд орхичих." Доорх задаргаа зөвхөн тэмдэглэл — идэвхтэй ажил ЭХЛЭХГҮЙ хойшлуулаагүй хүртэл.

- `DigitalSignature` component + `digital-signature-select-modal`.
- g-sign: `createSignRequest` → `sendRequestGSign(phone, pdfSignRequestId)` → socket (`getDigitalSignatureSocketUrl`) → status poll. e-mongolia/DAN redirect урсгал.
- `registerDigitalSignCert`, `getUrlByOtp`.
- **Үр дүн:** тоон гарын үсэг ажиллана. Effort: **xhigh**.

### ⏸️ 2d-4 (сонголт): Payment — ХОЙШЛУУЛСАН (2d-3-тэй хамт)
- pay товч → `getPaymentByContractRequest` + төлбөрийн урсгал. Тусдаа effort.

---

## Архитектурын шийдвэрүүд

| Шийдвэр | Сонголт | Учир |
|---|---|---|
| Дараалал | **2d-1 эхэлж** (verification), sign/digital хойш | 2d-1 хамаарал бага, шууд үнэ цэнэ. sign нь PDF viewer-т хоригдоно. |
| Verification modal | v2 `action-verification-modal` порт, shadcn Dialog + `input-otp` | v3-т `input-otp` бий (Phase 1 password modal). |
| OTP verify API | `sendVerificationRequest`/OTP endpoint — backend тулга | v2 lib-ээс нарийн endpoint авах. |
| Signing UI | pdf-viewer-new-тэй нэгтгэнэ | Талбар бөглөх нь viewer дээр. Тусдаа effort. |
| taskVerified | 2d-2-т DAN холбоно; 2d-1 хүртэл `true` хэвээр | 2c-ийн placeholder тайлбар. |

---

## v3 дээр бэлэн, дахин ашиглах

| Хэрэгцээ | Эх сурвалж | Замд |
|---|---|---|
| Mutation + invalidate | `useDeleteContract` г.м | `features/documents/api/queries.ts` |
| OTP input | `input-otp` (Phase 1 password modal) | `features/documents/components/PasswordChangeModal.tsx` |
| Dialog | shadcn `dialog.tsx` (2c FeedbackModal) | `components/ui/dialog.tsx` |
| ActionButtons placeholder prop-ууд | `ActionButtons.tsx` (2c) — onDelete/onResend/onAcceptCancellation/onSign/... | `features/documents/components/detail/ActionButtons.tsx` |
| PDF viewer | [[pdf-viewer-new]] | `shared/pdf-viewer` |

---

## Endpoint лавлагаа (v2-ээс)

| Үйлдэл | Method | Path / fn |
|---|---|---|
| accept-cancellation | POST | `/contract-action/accept-cancellation` `{contractRequestId, contractActionId}` |
| resend | POST | `/contract-request/resend` |
| delete | POST | `/contract-request/remove/{id}` (v3-т `deleteContract` бий) |
| approve (sign verify+finalize) | POST | `/contract-action/approve` `{verify, codes?, contractActionId, participantKey, contractRequestId, contractFieldList}` ✅ v3 портлогдсон |
| review | POST | `/contract-action/review` `{verify, codes?, contractActionId, contractRequestId}` ✅ v3 портлогдсон |
| signature create | POST | `signature.js`: `createSignature`, `signatureFileUpload` (`/file/upload`), `bgRemove`, `signatureHistoryList` |
| digital g-sign | GET/POST | `digitalSignature.js`: `createSignRequest`, `sendRequestGSign` (`/g-sign/contract/sign?phoneNumber=&pdfSignRequestId=`), `getDigitalSignatureSocketUrl`, `getUrlByOtp`, `registerDigitalSignCert` (base `DIGITAL_SIGNATURE_URL`) |

---

## v2 эх сурвалж (порт хийхэд)

- Verification modal: `components/custom/modals/action-verification-modal`, `pages/auth/sercurity-verification-modal`, `pages/auth/dan-verification-modal`
- Signature зурах: `components/custom/signature-input`, `context/signature-create-context.jsx`, `pages/profile/user/signature/*`
- Digital signature: `components/custom/digital-signature`, `components/custom/modals/digital-signature-select-modal`
- Sign submit flow: `pages/contract-detail/index.jsx` `onSubmit` (L348), `onSubmitReview` (L385), `handleApproveContract` (L547), `handleReviewContract` (L571)
- Sign payload: `{ verify:false, contractActionId, participantKey, contractRequestId, contractFieldList }`

---

## Verification (phase бүрд)

1. `npm run lint && npx tsc --noEmit && npm run knip`.
2. 2d-1: delete/resend/accept-cancellation товч → verification modal → OTP → амжилт, detail refetch, төлөв солигдоно.
3. 2d-2: sign/review/approve → талбар бөглөх → OTP → гарын үсэг зурагдаж гэрээ SIGNED болно.
4. 2d-3: тоон гарын үсэг → g-sign урсгал → DIGITAL_SIGNED.
5. `npm run test:e2e` — critical-flows.
6. Гар аргаар `npm run dev`.

---

## Гүйцэтгэлийн дараалал, модел

| # | Sub-phase | Хамаарал | Effort | Модел |
|---|---|---|---|---|
| 2d-1 | Verification modal + delete/resend/accept-cancellation | 2c | **high** | **Opus 4.8** ✅ дууссан |
| — | pdf-viewer view-only wiring | — | low | Sonnet ✅ дууссан |
| 2d-2a | Талбар ДИСПЛЕЙ (read-only, `fields` prop) | pdf-viewer view-only | **low** | **Sonnet** ✅ дууссан |
| 2d-2b | Талбар ФИЛЛ (SIGN_PENDING) | 2d-2a + type | ~~high~~ **low** (бодит) | ~~Opus~~ **Sonnet** ✅ дууссан |
| 2d-2c | OTP verify + submit finalize (sign/review/approve) | 2d-2b | ~~xhigh~~ **medium** (бодит) | ~~Opus~~ **Sonnet** ✅ дууссан |
| 2d-3 | Digital signature (g-sign/DAN) | 2d-2c | **xhigh** | **Opus 4.8** |
| 2d-4 | Payment (сонголт) | — | high | Opus/Sonnet |

**Дүрэм:** sub-phase бүр = ШИНЭ чат + verify+commit. Механик хэсэг (endpoint/hook/type өргөтгөл, pattern-following display) → Sonnet subagent; нарийн урсгалын логик (interactive edit, OTP/sign/socket) → Opus. caveman + ponytail хэвээр.

### Санал болгох эхлэл
2d-2a, 2d-2b, 2d-2c ✅ дууссан (commit `8dd7531`). **2d-3, 2d-4 хойшлогдсон** — хэрэглэгчийн шийдвэрээр "бүр дараа" хийнэ, одоохондоо зөвхөн доорх задаргаа тэмдэглэл хэлбэрээр үлдэнэ. Дахин эхлэхэд эхлээд: (1) 2d-2c-ийн OTP урсгалыг browser дээр гар аргаар баталгаажуулах (authenticated session + бодит SIGN_PENDING/REVIEW_PENDING гэрээ), дараа нь (2) 2d-3-аас үргэлжлүүлэх.

### Чат стратеги (токен)
- Sub-phase бүр ШИНЭ чат: seed = энэ note + тухайн v2 эх сурвалж + зорилтот v3 файл.
- graphify эхэлж → зорилтот файл л унших. Subagent-аар model солино (2c-тэй ижил waterfall).
