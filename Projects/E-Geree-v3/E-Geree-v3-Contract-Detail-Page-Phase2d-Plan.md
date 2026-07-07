---
title: "Plan: Contract Detail Page — Phase 2d (signing + 2FA + digital signature)"
type: project
status: in-progress
created: 2026-07-03
updated: 2026-07-07
tags:
  - project
  - project/e-geree-v3
  - plan
---

# Plan: Contract Detail Page — Phase 2d (гарын үсэг + 2FA + тоон гарын үсэг)

> [!warning] Аудит 2026-07-07 (Claude) — дутуу зүйлс
> **Ерөнхий төлөв:** mostly-done (13/18 хэрэгжсэн)
> **Дутуу / хийгдээгүй:**
> - 🟡 GSIGN live runtime-verify — DIGITAL_SIGNATURE_PENDING гэрээ + DAN push утас шаардана; static+build code-confidence л хүрсэн (plan мөр 155, 187-194; Worklog-2026-07-07 "NOT browser-drive-verified")
> - 🟡 TRIDUM live runtime-verify — гэрээ + суусан ws://127.0.0.1:59001 desktop клиент шаардана; a710337 unit test-тэй ч жинхэнэ WS клиент тестийн тэмдэглэл алга (plan мөр 169)
> - 🟡 Payment live runtime-verify — paymentType:PAY + paymentStatus:PENDING гэрээ + QPay/банк sandbox шаардана; ff09d0b static verify PASS л гэсэн (plan мөр 193)
> - ⚪ Deploy/ops: prod-д DIGITAL_SIGNATURE_URL, NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL, PAYMENT_URL_V2 env inject батлагдаагүй — эдгээр var repo-ийн ямар ч CI/Dockerfile-д алга (plan мөр 157 "Үлдэц = ops, код биш")
> - ⚪ npm run test:e2e (critical-flows) 2d фазуудад ажилласан нотолгоо алга — playwright-report/test-results mtime 2026-06-30, эхний 2d commit-оос өмнөх
> **Тэмдэглэл:** Кодын бүх ажил дууссан — 10 commit бүгд dev-khishigee дээр батлагдав (2d-1…2d-4, OTP live-verify PASS); үлдсэн нь live runtime-verify + ops. Frontmatter status: draft хэвээр байгаа нь агуулгатайгаа зөрчилдөж байгаа тул шинэчлэх нь зүйтэй.

**Огноо:** 2026-07-03
**Салбар:** dev-khishigee
**Төлөв:** 📝 Ноорог (батлагдаагүй)
**Холбоотой:** [[E-Geree-v3-Contract-Detail]] (2a plan устгагдсан) · [[E-Geree-v3-Contract-Detail-Page-Phase2c-Plan]] · [[E-Geree-v3-PDF-Viewer]] · [[E-Geree-v3-Networking-BFF]]

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
| DAN verified flag (`authUser.danVerified`) | taskVerified | ✅ ШИЙДЭГДСЭН (2026-07-07 audit): `taskVerified` нь `auth-types.ts:61` **dead type талбар** — documents feature-д огт ашиглагддаггүй. v3-т client-side sign-gate ОГТ БАЙХГҮЙ (`action-status.ts` зөвхөн `actionData.status`-аар gate хийнэ). Баталгаажуулалтыг **backend** хийдэг (`methods`/error). "2c hardcoded true" = stale, portлогдоогүй. Хийх зүйл алга. |

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
- **Үр дүн:** sign/review OTP хүртэл бүрэн ажиллана.
- **Дараагийн алхам:** 2d-3 (digital signature, g-sign/DAN) — 2d-2c-аас хамааралтай, одоо нээлттэй.

**🔬 Browser-verify (2026-07-06, `dev-khishigee`, live prod backend `public-api.e-geree.mn` — staging алга):** `verify` skill-ээр гар аргаар жолоодов. Гэрээ `6a4b5dee…622d` ("Test", SIGN_PENDING, өөрийн 2 талт test).
- ✅ `fieldsIncomplete` guard: талбар дутуу үед Approve дарахад **сүлжээ хүсэлт гарахгүй** (fill tab руу).
- ✅ 2d-2b fill/signature preset: "Гарын үсэг зурах" → `signatureImgUrl` apply → PDF live preview.
- ✅ `verify:false` payload ЯГ зөв: `POST /backend/public-api/v1/contract-action/approve` `{verify:false, participantKey:"participant2", contractRequestId, contractActionId, contractFieldList:[7 талбар]}`. Response handling (apiClient unwrap), `onSuccess` conditional-invalidate, detail refetch, PDF `temporary-contract→contract`, түүх "гарын үсэг зурсан", 2 оролцогч ✓ — БҮГД ажиллана. Гэрээ **SIGNED** боллоо.
- ⚠️ **~~ГОЛ НЭЭЛТ — DAN нь OTP-г алгасдаг~~ — 2026-07-07 live-verify-д БУРУУ нь батлагдав (доор үз): `methods` = хэрэглэгчийн идэвхжүүлсэн EMAIL/SMS/2FA-аар gate, `danVerified`-тай хамаагүй. Энэ account-д арга идэвхгүй байсан тул л шууд SIGNED болсон.** ~~энэ хэрэглэгч `userVerifiedType:"DAN_VERIFIED"`. Backend `verify:false`-д **`methods` буцаадаггүй**, шууд SIGNED болгодог. Тиймээс `!data.methods` салаа шатдаг → `SecurityVerificationModal` **огт нээгддэггүй**. Өөрөөр хэлбэл 2d-2c-ийн ГОЛ (OTP `methods`→modal→`verify:true`+codes→wrong-code error) урсгал **DAN хэрэглэгчид хүрэшгүй** — зөвхөн **non-DAN оролцогчид** (email/SMS OTP) л ажиллана. Мөн DAN хэрэглэгчид `verify:false` нь өөрөө destructive (шууд гарын үсэг зурдаг, dry-run алга).~~
- **Дүгнэлт:** хүрэх боломжтой бүх зам ✅ PASS; OTP branch = зөвхөн code-review confidence (login-2FA two-phase pattern-тэй ижил, батлагдсан). Хэрэглэгч "accept code-confidence" сонгов. Жинхэнэ OTP урсгалыг шалгахад **non-DAN test account** хэрэгтэй.
- ⚠️ **Санамсаргүй жинхэнэ гарын үсэг:** wrong-OTP probe төлөвлөсөн ("гарын үсэг үүсэхгүй") боловч DAN OTP алгассан тул гэрээ бодитоор SIGNED боллоо. Хохирол бага (өөрийн test, 2 тал өөрөө). 2d-3 (digital/DAN) ба цаашид энэ account дээр аливаа sign тест = жинхэнэ SIGNED гэдгийг санах.

**🔬 non-DAN OTP LIVE-VERIFY — DUUSSAN ✅ (2026-07-07, chrome-devtools, хувь хүн профайл `khishigbayar.u`, EMAIL+2FA идэвхжсэн; гэрээ `6a0fcf9c…d473` "Sender - Employee transfer or delete test", participant2 SIGN_PENDING):**
- 🔑 **ЗАЛРУУЛГА (өмнөх DAN-загвар буруу):** `methods` нь хэрэглэгчийн **идэвхжүүлсэн verification-оор** (EMAIL/SMS/2FA) gate хийгддэг, `danVerified`-тай ХАМААГҮЙ. Нотолгоо: энэ account `userVerifiedType:"DAN_VERIFIED"` мөртлөө EMAIL+2FA идэвхтэй тул `methods:{EMAIL:true,"2FA":true}` буцаалаа. `action.userVerifiedType:"UN_VERIFIED"` ≠ non-DAN дохио (андуурч болохгүй).
- ✅ approve `verify:false` → `{requireAuthorization:true, methods:{EMAIL:true,"2FA":true}, expireDate}`; SIGNED болоогүй (detail refetch гараагүй).
- ✅ `SecurityVerificationModal` нээгдэв: 2 талбар (И-мэйл + 2FA, label зөв), countdown 274s→ ажиллав.
- ✅ `verify:true`, `codes:{EMAIL,"2FA"}` dynamic multi-method payload зөв → `status:"SIGNED"` → success invalidate → detail refetch, addition-file, PDF `temporary→contract`, түүх, rail→meta.
- ⚠️ Тестлээгүй (хэрэглэгч зөв кодоор шууд дуусгасан): буруу-код backend-reject error branch + `retrySendCode` — code-confidence хэвээр.
- **Дүгнэлт:** OTP submit урсгал (methods→modal→verify:true+codes→SIGNED) **бүрэн live-PASS**. Гэрээ бодитоор SIGNED.

### ✅ 2d-3 (GSIGN): Digital signature — DUUSSAN (2026-07-07, commit `1ac18a2`)
**Хэрэглэгчийн шийдвэр (2026-07-07):** хамрах хүрээ = **GSIGN only** (DAN/e-Mongolia утасны push); TRIDUM (суусан desktop клиент) → **2d-3b** хойшлуулав. Тест = **code-confidence + UI-хүртэл** (жинхэнэ гарын үсэг гүйцээхгүй).

**Understand-workflow (6 parallel reader) гол нээлтүүд:**
- ⚠️ **Plan хуучирсан:** `FlowActionDock.tsx` (commit `9e6409b`) нь `c2c07e5`-д УСТСАН, `ActionButtons.tsx`-д нэгтгэгдсэн. Зорилтот = `ActionButtons` + `action-status.ts`. `FlowActionDock` дахин үүсгэхГҮЙ.
- ✅ `onDigitalSign` hook аль хэдийн бэлэн (`ActionButtons.tsx:240-245` flowClick, default noop). `getFlowAction` digitalSign kind буцаана. `ActionButtons` **засвар хэрэггүй** — зөвхөн ContractDetail-д handler дамжуулна.
- ✅ **Backend v2-тэй ИЖИЛ хост** (`digital-signature-api.e-geree.mn/digital-signature-api/v1`) → v2 контракт = эх сурвалж, протокол-эрсдэл бага (approve/review шиг).
- ✅ **Envelope faithful port:** `coreFetcher` `{isOk: res.ok, data: rawBody}` (HTTP статусаар), v2 `httpRequest`-тэй ижилхэн → v2-т `data.otp` уншсан бол v3-т unwrap-ийн дараа `.otp`. Микросервис ялгаагүй.
- ⚠️ **v3-т socket infra ОГТ алга** — scratch-аас барьсан. `socket.io-client@^4` суулгав (v2 = `^4.7.5`, server socket.io — major тааруулав).
- Env бэлэн (утаслагдаагүй байсан): `DIGITAL_SIGNATURE_URL` (base version-baked), `NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL`. Утас = `user.userProfile.mobile` (top-level `phone` биш).

**Барьсан (7 өөрчлөлт, GSIGN):**
- `core/config/index.ts`: `getDigitalSignatureUrl()` (getPaymentUrl-ийн хуулбар, E2E override).
- **New** `app/backend/digital-signature/[...path]/route.ts` (payment route хуулбар, POST+GET). Route folder = URL segment, `[...path]` upstream руу форвардлана; base version-baked тул path давхар version нэмэхгүй.
- `documents/api/client.ts`: `createSignRequest` (POST `/digital-signature/pdf-sign-request/create`), `sendGSignRequest` (GET `/g-sign/contract/sign?...`) + `CreateSignRequestPayload`/`SignRequestResult` types.
- `documents/api/queries.ts`: `useStartGSign` — create+g-sign-ийг **нэг мутацид нэгтгэв**, буцаах утга = socket room key (`pdfSignRequestId`). `otp/inputPdfUrl/id` дутуу бол throw (v2-ийн `data.otp && data.inputPdfUrl` guard-ийн порт). invalidate ЭНД биш — socket `sign`-ийн дараа ContractDetail хийнэ.
- **New** `documents/components/detail/DigitalSignModal.tsx` — GSIGN modal: утас (userProfile.mobile-аас preset) → push → waiting (socket wait) → success/error. shadcn Dialog + raw Монгол текст (detail-ийн house-style, i18n нэмэлтгүй). **v2-ийн 2 bug зассан:** (1) module-level socket leak → useEffect cleanup `socket.disconnect()`; (2) transport failure дээр мөнхөд өлгөгдөх → `connect_error` handler нэмэв. `onSignedRef` (render-т ref бичихгүй, effect-т) reconnect churn-аас сэргийлнэ.
- `ContractDetail.tsx`: `handleDigitalSign` (modal нээх) + `handleDigitalSigned` (socket амжилтад `queryClient.invalidateQueries(detail)` + toast + хаах); `onDigitalSign={handleDigitalSign}` дамжуулав; `<DigitalSignModal>` render.

**Socket контракт (v2 порт):** `io(\`${base}/digital-signature\`, {transports:['websocket'], withCredentials:true})` → `connect` дээр `emit('join-room',{room:pdfSignRequestId})` → `on('sign', JSON.parse→{result,message})`. success=result; `"User cancelled the request"`→татгалзсан; бусад→алдаа.

**eslint нээлт:** (1) render үед `ref.current=` бичих нь `react-hooks/refs` error → effect руу. (2) useEffect дотор синхрон setState = `react-hooks/set-state-in-effect` (3x) → **render-т compare-and-set** (`prevOpen` transition, ContractDetail-ийн `seededFieldList` idiom-тэй ижил); socket-base check submit руу зөөв.

**Verify:** tsc 0 · eslint 0 (шинэ/өөрчилсөн файл; зөвхөн pre-existing `CopyIconButton` warning ContractDetail-д, 2d-2c-ийн, минийх биш) · knip шинэ мэдэгдэлгүй · vitest 28/28 · **`npm run build` exit 0** (route бүртгэгдсэн, socket.io-client bundle/SSR цэвэр).
**Хийгдээгүй runtime-verify:** live UI-drive DAN push хүртэл — `DIGITAL_SIGNATURE_PENDING` статустай гэрээ шаардана (одоо бэлэн алга; гүйцээвэл ЖИНХЭНЭ SIGNED). Code-confidence (v2 faithful port + ижил backend) хүрсэн. **DEPLOY санамж:** `DIGITAL_SIGNATURE_URL`/`NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL` env prod-д хэрэгтэй (.env gitignored).

**✅ Env audit (2026-07-07):** `.env.production` + `.env.development` локалд аль хэдийн зөв утгатай (`DIGITAL_SIGNATURE_URL=.../digital-signature-api/v1`, `NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL=https://digital-signature-socket.e-geree.mn`). Хоёулаа gitignored (commit хийгдээгүй) тул `npm run build` цэвэр байсан нь локал файлаас. **Үлдэц = ops, код биш:** deploy target (Docker/CI) эдгээр 2 var-г файлаар унших/inject хийхийг батлах — кодоос шалгах боломжгүй.

### ✅ 2d-3b: Digital signature (TRIDUM) — DUUSSAN (2026-07-07, static+unit verify PASS; commit `a710337`)
Суусан desktop гарын үсгийн клиент (raw WebSocket `ws://127.0.0.1:59001`). GSIGN-ээс тусдаа урсгал: version probe (`{Command:'99'}`) → `{Command:'4', OTP, SignLocation, PDFFiles}` → `getUrlByOtp` + `registerDigitalSignCert`. Location calc = `DIGITAL_SIGNED` action тоогоор.

**Барьсан (5 өөрчлөлт, v2 `use-tridum.js` + `digital-signature-select-modal` faithful порт):**
- `api/client.ts`: `CreateSignRequestPayload.type` → `"GSIGN" | "TRIDUM"`; шинэ `getSignedUrlByOtp` (GET `/signed-pdf/url-by-otp`), `registerDigitalSignCert` (POST `/signed-pdf/save-serial-number`). BFF POST+GET аль хэдийн хангасан.
- **New** `lib/tridum.ts`: `calcTridumLocation(actionList)` — 2 баганаар (тэгш→зүүн X0.1, сондгой→баруун X0.55; мөр Y+0.1) шатлана. `ponytail: pdfPageNumber v3-т үргэлж null → base page 1`. + `tridum.test.ts` (4 байрлал, toBeCloseTo — FP).
- **New** `components/detail/use-tridum-sign.ts`: localhost WS hook. `getVersionStatus` (`{Command:'99'}` 1s probe → шинэ клиент бол PageNumber хасахгүй) → `{Command:'4'}` sign → response parse (`SignResult===0` эсвэл 1–10/`7:regNum` error-код Монгол текст). v2-оос сайжруулсан: unmount-д `wsRef` хаана.
- `components/detail/DigitalSignModal.tsx`: method radio (GSIGN/TRIDUM) нэмэв. GSIGN = хуучин утас+socket урсгал (method guard-тай); TRIDUM = `createSignRequest(type:TRIDUM)` → `sendTridumSignRequest` → `onResponse`-д `getSignedUrlByOtp`+`registerDigitalSignCert` → `onSigned`. `actionList` prop нэмэв.
- `components/ContractDetail.tsx`: `actionList={data.actionList}` дамжуулав.

**Verify:** tsc 0 · eslint 0 (hook нэр kebab `use-tridum-sign.ts` болгосон, check-file дүрэм) · vitest 30/30 · knip шинэ флаггүй · `npm run build` OK.
**Хийгдээгүй runtime-verify (2d-3-тэй ижил хязгаар):** (1) UI-drive — `DIGITAL_SIGNATURE_PENDING` гэрээ шаардана (одоо алга); (2) TRIDUM WS — суусан `ws://127.0.0.1:59001` десктоп клиент шаардана (энд алга). Тиймээс code-confidence (v2 faithful порт + ижил backend). Гүйцээвэл ЖИНХЭНЭ DIGITAL_SIGNED.

### ✅ 2d-4: Payment — DUUSSAN (2026-07-07, static verify PASS; commit `ff09d0b`)
pay товч → төлбөрийн урсгал. v2 = глобал `PaymentProvider` context (subscription+template+contract 3 модалд) — v3 contract-д хэт эвдэрхий тул **self-contained modal** (нэг л хэрэглэгч, context барихгүй — ponytail).

**Барьсан (6 өөрчлөлт):**
- `core/config`: **New** `getPaymentUrlV2()` — v2 `payment/initiate` л /v2-т байдаг. `PAYMENT_URL` нь /v1-ээр baked тул /v2 руу regex-ээр сольж гаргана; `PAYMENT_URL_V2` env байвал эрхэмлэнэ (`ponytail: тусдаа env-гүй локалд ажиллана, deploy override сонгол`).
- **New** `app/backend/payment-v2/[...path]/route.ts` — /v2 BFF (POST+GET; initiate=POST).
- `documents/api/client.ts`: `getPaymentByContractRequest` (дүн), `getPaymentMethodList`, `paymentInitiate` (/payment-v2), `checkPaymentStatus` + type-ууд (`PaymentInfo`/`PaymentMethod`/`PaymentInitiatePayload`/`PaymentInitiateResult`).
- **New** `components/detail/ContractPaymentModal.tsx` — getPaymentByContractRequest+method жагсаалт (useQuery) → арга сонгох → initiate → **QPay** бол QR(base64)+deeplink + payment socket (`${base}/payment` room=`number`) `pending`→success; **redirect** арга (SOCIAL_PAY/BANK_CARD/ARD) бол шинэ таб (popup blocked→ижил таб); "Төлбөр шалгах" товч = `checkPaymentStatus` (conservative truthy — money-path, эргэлзвэл socket эрхэмлэнэ). НӨАТ баримт = оролцогчийн төрлөөс авто.
- `ContractDetail.tsx`: `onPay={handlePay}` (ActionButtons-ийн бэлэн placeholder-руу) + `handlePaid` (invalidate+toast+хаах) + modal render.

**Verify:** tsc 0 · eslint 0 · vitest 30/30 · knip шинэ флаггүй · `npm run build` OK (payment-v2 route бүртгэгдсэн).
**Хойшлуулсан (ponytail, эрэлтээр):** NuatSection (НӨАТ баримтын төрлийг ГАРААР солих UI — одоо профайлаас авто); redirect методын popup-blocker gymnastics (v2 about:blank pre-open) хялбарчилсан; subscription/public-template төлбөр (энэ фазын хамрах хүрээнд биш).
**DEPLOY санамж:** `PAYMENT_URL_V2` prod-д тодорхойлох (эсвэл `PAYMENT_URL` нь `/v1`-ээр төгсөж байгаа эсэхийг батлах — regex fallback түүнд түшиглэдэг).

---

## 🔬 Live-verify pending (орчин бэлдсэний дараа — 2026-07-07 хэрэглэгчийн санамж)
GSIGN, TRIDUM, Payment гурвуулаа **static+build code-confidence** төвшинд DUUSSAN; жинхэнэ runtime-verify нь тусгай орчин шаардана:
| Үйлдэл | Live-test-д хэрэгтэй орчин |
|---|---|
| **GSIGN** | `DIGITAL_SIGNATURE_PENDING` статустай гэрээ + DAN утасны push хүлээн авах утас |
| **TRIDUM** | Дээрхийн адил гэрээ + суусан `ws://127.0.0.1:59001` десктоп клиент |
| **Payment** | `paymentType:"PAY"` + `paymentStatus:"PENDING"` гэрээ + QPay/банк sandbox + `PAYMENT_URL_V2` env |
Орчин бэлэн болмогц non-DAN OTP-той ижил байдлаар chrome-devtools-оор жолоодож батална (бүгд ЖИНХЭНЭ гүйлгээ/гарын үсэг үүсгэнэ гэдгийг санах).

---

## Архитектурын шийдвэрүүд

| Шийдвэр | Сонголт | Учир |
|---|---|---|
| Дараалал | **2d-1 эхэлж** (verification), sign/digital хойш | 2d-1 хамаарал бага, шууд үнэ цэнэ. sign нь PDF viewer-т хоригдоно. |
| Verification modal | v2 `action-verification-modal` порт, shadcn Dialog + `input-otp` | v3-т `input-otp` бий (Phase 1 password modal). |
| OTP verify API | `sendVerificationRequest`/OTP endpoint — backend тулга | v2 lib-ээс нарийн endpoint авах. |
| Signing UI | pdf-viewer-new-тэй нэгтгэнэ | Талбар бөглөх нь viewer дээр. Тусдаа effort. |
| taskVerified | ~~2d-2-т DAN холбоно~~ → **холбохгүй** (2026-07-07 audit): dead type талбар, client gate байхгүй, backend enforce | v3-т client-side sign-gate портлогдоогүй — зөв дизайн. |

---

## v3 дээр бэлэн, дахин ашиглах

| Хэрэгцээ | Эх сурвалж | Замд |
|---|---|---|
| Mutation + invalidate | `useDeleteContract` г.м | `features/documents/api/queries.ts` |
| OTP input | `input-otp` (Phase 1 password modal) | `features/documents/components/PasswordChangeModal.tsx` |
| Dialog | shadcn `dialog.tsx` (2c FeedbackModal) | `components/ui/dialog.tsx` |
| ActionButtons placeholder prop-ууд | `ActionButtons.tsx` (2c) — onDelete/onResend/onAcceptCancellation/onSign/... | `features/documents/components/detail/ActionButtons.tsx` |
| PDF viewer | [[E-Geree-v3-PDF-Viewer]] | `shared/pdf-viewer` |

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
| 2d-3 | Digital signature (**GSIGN** / DAN) | 2d-2c | ~~xhigh~~ **medium** (бодит) | Opus (ultracode workflow) ✅ дууссан |
| 2d-3b | Digital signature (TRIDUM, суусан клиент) | 2d-3 | high | ✅ дууссан (2026-07-07, static verify) |
| 2d-4 | Payment (сонголт) | — | high | ✅ дууссан (2026-07-07, static verify) |

**Дүрэм:** sub-phase бүр = ШИНЭ чат + verify+commit. Механик хэсэг (endpoint/hook/type өргөтгөл, pattern-following display) → Sonnet subagent; нарийн урсгалын логик (interactive edit, OTP/sign/socket) → Opus. caveman + ponytail хэвээр.

### Санал болгох эхлэл
2d-2a, 2d-2b, 2d-2c ✅ (commit `8dd7531`). **2d-3 GSIGN ✅ дууссан** (2026-07-07, commit `1ac18a2`; static+build verify PASS, live UI-drive хийгээгүй).

**Үлдэгдэл дараалал (2026-07-07 санал, эрсдэл × өртөг):**
1. ~~Prod env утаслах~~ ✅ audit: локал зөв, gitignored; deploy inject батлах (ops).
2. ~~`taskVerified` тулга~~ ✅ audit: dead field, gate байхгүй, backend enforce. Хийх зүйл алга.
3. ~~**non-DAN OTP live-verify**~~ ✅ DUUSSAN (2026-07-07): EMAIL+2FA идэвхжсэн account-аар methods→modal→verify:true+codes→SIGNED бүрэн live-PASS. Залруулга: methods = enabled verification-оор gate, danVerified-тай хамаагүй. (Үлдэц: буруу-код/retry branch = code-confidence.)
4. ~~**2d-3b (TRIDUM)**~~ ✅ DUUSSAN (2026-07-07, static verify) · ~~**2d-4 (Payment)**~~ ✅ DUUSSAN (2026-07-07, static verify).
5. **Live-verify pending (орчин бэлдсэний дараа):** GSIGN / TRIDUM / Payment — дээрх "🔬 Live-verify pending" хүснэгт үз.
- signature-input UI: preset (`signatureImgUrl`) хангаж байгаа тул YAGNI, эрэлт гарвал.

### Чат стратеги (токен)
- Sub-phase бүр ШИНЭ чат: seed = энэ note + тухайн v2 эх сурвалж + зорилтот v3 файл.
- graphify эхэлж → зорилтот файл л унших. Subagent-аар model солино (2c-тэй ижил waterfall).
