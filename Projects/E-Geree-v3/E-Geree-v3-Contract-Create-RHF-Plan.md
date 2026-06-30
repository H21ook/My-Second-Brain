---
title: "Contract-Create — react-hook-form интеграцийн төлөвлөгөө"
type: project
status: done
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - plan
  - project/e-geree-v3
---

# Contract Create — react-hook-form интеграцийн төлөвлөгөө

> [!note] Хэрэгжүүлэх төлөвлөгөө. Бодит код дээр баталгаажуулсан (2026-06-30); нээлттэй байсан 2 шийдвэр (draft sync timing, Next-gate) шийдсэн.

[[E-Geree-v3-Contract-Create-Feature]] · холбоотой: [[E-Geree-v3-State-Management]]

## Context — яагаад

Гэрээ үүсгэх wizard-ийн form control **react-hook-form ашигладаггүй** (RHF зөвхөн `login-form.tsx`-д л бий). Бүх 4 алхам Redux-аар гараар удирдагдаж байгаа нь form ergonomics дутуу мэдрэгдэх үндсэн шалтгаан:

- **State**: input бүр товчлуур дарах болгонд Redux руу `dispatch` (`updateParticipant`, `updateField`, `setTitle`...). Input бүр Redux-ээс уншиж keystroke бүрт re-render үүсгэдэг.
- **Validation**: алхам бүрд нэг том гар бичсэн `validateStep()` зөвхөн **"Үргэлжлүүлэх" дарахад** ажиллана (`lib/validation-steps.ts:116`). Талбар тус бүрийн onBlur/onChange валидаци, `touched`/`dirty` алга.
- **Error төлөв 3 газар тарсан**: (1) `WizardShell` local `errors` + `WizardErrorsProvider` (`WizardShell.tsx:55`, `wizard-errors.ts`), (2) `SubmitStep` доторх тусдаа `useState` errors (`SubmitStep.tsx:82`), (3) `fields.slice.errors`.
- **Async validation** (компани/гишүүн шалгах) бүгд гараар.

**Гол хүндрэл:** Wizard-ийн алхам бүр **тусдаа Next.js route** (`/contract/create/{content,participant,fields,submit}`). Алхам солиход step page **unmount** болно; `WizardShell` + `WizardBootstrap` нь `layout.tsx`-д тул хэвээр. Иймд алхамд шууд `useForm` тавихад navigation хийхэд RHF-ийн in-memory төлөв **алга болно**. Алхам хооронд амьд үлддэг нь **Redux + localStorage draft + IndexedDB blob** л.

**Зорилго:** Form-уудыг **react-hook-form + Zod** руу шилжүүлж per-field validation, нэгдсэн error загвар, цэвэр async урсгал авах — Redux/persist давхаргыг **эвдэхгүйгээр**.

---

## Архитектурын шийдвэр

1. **Per-step `useForm`** (алхам бүрд нэг; дэд хэсэгт `FormProvider` + `useFormContext`). Single mega-form хийхгүй — route-ууд тусдаа, Fields canvas RHF-д таарахгүй.
2. **Redux = алхам хоорондын source of truth** (persist/IndexedDB хэвээр). **RHF = алхам доторх source of truth** (засвар + validate + per-field error). Хооронд нь гүүрээр холбоно.
3. **Hybrid** — зөвхөн form input бүхий алхам RHF. **Fields canvas (drag/drop, undo, groupId/order)** Redux хэвээр; зөвхөн `FieldSettingsPanel`-ийн input RHF.

---

## 1. RHF ↔ Redux гүүр (3 хэсэг)

**(a) Унших — `defaultValues` ← Redux selector.** Алхам mount бүрт Redux-ээс анхны утга (Redux survive хийсэн тул мэдээлэл хадгалагдана). Ж: ContentStep ← `selectContent`, ParticipantStep ← `selectParticipants` (`store/selectors.ts`).

**(b) Бичих (draft) — `form.watch` subscription → debounce ~1000ms → `dispatch`.**
- `useEffect(() => { const sub = form.watch(v => debouncedDispatch(v)); return () => sub.unsubscribe() })`.
- Render-д `form.watch()`-ийн буцаах утгыг ашиглахгүй (subscription хэлбэр) → нэмэлт re-render үүсэхгүй.
- Доош нь одоогийн `persist.ts` 500ms throttle хэвээр → localStorage бичилт өмнөхөөс олон болохгүй.
- **Ачаалал:** RHF `register` input нь uncontrolled тул keystroke бүрт re-render **арилна** (одоогийнхоос **хөнгөн**). Sync зөвхөн draft-д хэрэгтэй учир 1000ms тэнцвэртэй (500ms = илүүц; 5s = crash үед хэт их алдах).

**(c) Navigation gate — Gate-registry context.**
- Жижиг context (`WizardStepGate`): идэвхтэй алхам mount дээрээ `registerGate(async () => form.trigger())` бүртгэнэ.
- `WizardShell.handleNext` нь `validateStep(...)`-ийн оронд gate дуудна: `const ok = await gate(); if (!ok) return; <cleanup>; goNext()`.
- WizardShell төв зохион байгуулагч хэвээр; Content-алхмаас гарах "сонгоогүй төрөл цэвэрлэх" логик (`WizardShell.tsx:96-107`) байрандаа.
- Алдаа inline нь RHF `formState.errors`-оор (context-оор биш); `form.trigger()` error-уудыг талбар бүрд автоматаар тавина.

---

## 2. Zod схемийг бодит болгох (нэг эх сурвалж)

`schema/index.ts`-ийн skeletal схемүүд runtime-д ашиглагддаггүй. Бодит дүрэм болгож `zodResolver`-аар холбоно. **Логик давхардуулахгүй** — helper-уудыг `superRefine` дотроос дуудна:
- `EMAIL_REG`, `REGISTRY_REG`, `isValidUsername`, `isVerificationValid` ← `lib/participants.ts`
- `checkRequiredFields` ← `lib/fields.ts` (FORM_FILLER, SIGNATURE/STAMP, SELECT options)
- `isTemplateFlow(createType)` ← `lib/create-type-helpers.ts`

**createType-aware (strict/relaxed):** schema-д `isTemplate` флаг (schema factory эсвэл `useForm` context). Template flow-д username + field value шаардахгүй дүрмийг `superRefine` дотор салгана. Ингэснээр `validation-steps.ts`-ийн бүх дүрэм Zod руу нэг эх сурвалжаас шилжинэ.

---

## 3. Алхам тус бүрийн зураглал

| Алхам | RHF хэрэгсэл | Тэмдэглэл |
|---|---|---|
| **ContentStep** | `useForm<ContentSchema>` | title/content + config switch. PDF upload Redux/IndexedDB хэвээр. **Pilot.** |
| **SubmitStep** | `useForm` | илгээгчийн бөглөх талбарууд; `handleTrySubmit` + local errors-ийг RHF орлоно; алдаа руу scroll хэвээр. Submit товч/AlertDialog алхамдаа (gate-д үл хамаарна). |
| **ParticipantStep** | `useForm` + `useFieldArray` | оролцогч массив + nested config; async гишүүн шалгалт (`useCompanyEmployeeCheck`)-ийг `setValue`/`setError`-оор. `keyName: "participantKey"`. **Хамгийн хүнд.** |
| **FieldSettingsPanel** | жижиг `useForm` | сонгосон талбарын props; submit дээр `updateField`/`updateGroupFields` dispatch. |
| **FieldsStep canvas** | — | Redux хэвээр; зөвхөн settings input RHF. |

**House pattern (лавлах):** `src/components/custom/login/login-form.tsx` — `useForm` + `zodResolver` + `register()` + `<Field>/<FieldError>` (`components/ui/field.tsx`) + React Query mutation. Шинэ shadcn `Form` primitive **нэмэхгүй** (`components/ui/form.tsx` байхгүй) — login-form паттернийг хуулбарлана.

---

## 4. Юу устгах / орлуулах (бүх алхам дууссаны дараа)

- `lib/validation-steps.ts` — Zod схемүүдээр орлуулагдана.
- `components/wizard-errors.ts` (`WizardErrorsProvider`/`useFieldError`/`useClearError`) — RHF `formState.errors`-оор.
- `WizardShell` local `errors`/`clearError` — gate + RHF болохоор устана.
- Алдааны key (`${participantKey}:field`, `select:${id}`) → RHF талбарын зам (`participants.0.username`, `fields.<id>.value`). Auto-select логик `formState.errors`-оос.

---

## 5. Шилжүүлэх дараалал (incremental — алхам бүр тусдаа commit)

1. **Суурь + ContentStep (pilot)** — gate-registry context, RHF↔Redux bridge helper, `contentSchema` бодит болгох, `WizardShell.handleNext` → gate. Паттерн тогтооно.
2. **SubmitStep** — бөглөх талбар + required.
3. **ParticipantStep** — `useFieldArray` + async гишүүн шалгалт.
4. **FieldSettingsPanel**.
5. **Цэвэрлэгээ** — `validation-steps.ts` + `wizard-errors.ts` устгана.

Шилжилтийн үед алхам бүрийг бүрэн дуусгаж merge → давхар source-of-truth-ээс сэргийлнэ.

---

## Critical files

- `components/WizardShell.tsx` — handleNext → gate; local errors устгах.
- `schema/index.ts` — Zod схемүүд бодит (createType-aware).
- `lib/validation-steps.ts`, `components/wizard-errors.ts` — эцэст нь устгана.
- `components/steps/ContentStep.tsx`, `SubmitStep.tsx`, `ParticipantStep.tsx`, `steps/participant/*`, `steps/fields/FieldSettingsPanel.tsx` — RHF болгох.
- `store/persist.ts` — **хөндөхгүй** (bridge энэ давхаргыг ашиглана).
- Reuse: `lib/participants.ts`, `lib/fields.ts`, `lib/create-type-helpers.ts`, `store/selectors.ts`, `components/ui/field.tsx`, `custom/login/login-form.tsx`.

---

## Эрсдэл ба tradeoff

- Шилжилтийн үед зарим алхам RHF, зарим Redux — **давхар source-of-truth**. Алхам бүрийг бүрэн дуусгана.
- `useFieldArray` key тогтворжилт (`keyName: "participantKey"`).
- Draft sync timing — `watch` → 1000ms debounce; доорх 500ms persist throttle-тэй уялдуулна.
- RHF↔Redux render давхардал — `watch`-ийг subscription хэлбэрээр, `useWatch`-г зөвхөн шаардлагатай талбарт.

---

## Rollback

Per-step additive тул алхам бүрийг буцааж болно.

---

## Шалгах (verification)

- Алхам бүрийн дараа: `npx tsc --noEmit` + `npm run lint` цэвэр.
- Playwright E2E (`npm run test:e2e`) wizard урсгалд унахгүй.
- **Draft parity**: бөглөөд (1s) refresh → утга сэргэх (localStorage + IndexedDB). Back/next → утга хадгалагдах.
- **Validation parity**: `validation-steps.ts`-ийн дүрэм бүр (title, ≥2 оролцогч, username, SIGN verification, required fields, template relaxed) Zod дээр давтагдсан эсэх.
- **Per-field UX**: талбар засахад алдаа арилах; "Үргэлжлүүлэх" дарахад алдаатай талбар тодрох.
- Submit happy-path: payload бүтэн (`payload.ts` Redux-ээс уншсан хэвээр), createType-strategy зөв endpoint руу.

---

## Гүйцэтгэл — хэрэгжүүлсэн (2026-06-30)

> [!success] 5/5 алхам бүрэн хэрэгжсэн. Branch: `feat/contract-create-rhf` (← `dev-khishigee`). Алхам бүр тусдаа commit; бүрд `npx tsc --noEmit` + `npm run lint` цэвэр (зөвхөн pre-existing 2 warning: `lib/payload.ts`, `shared/pdf-viewer/PdfViewer.tsx`).

### Гүйцэтгэлийн арга
Алхам бүрийг model/effort-аар нь subagent-д даалгаж, main loop (opus) дээр diff review → tsc/lint → commit → tracker шинэчлэлт хийв. **Тэмдэглэл:** Agent tool-д per-subagent effort knob байхгүй тул subagent-ууд session effort (high)-г өвлөв; model-ийг subagent бүрд override хийв.

### Алхмууд (commit / model)

| # | Алхам | Model | Commit |
|---|---|---|---|
| 1 | RHF суурь + ContentStep (pilot) | opus | `71a7c50` |
| 2 | SubmitStep RHF | sonnet | `8b62d5b` |
| 3 | ParticipantStep RHF + useFieldArray | opus (+main wiring) | `ab79afe` |
| 4 | FieldSettingsPanel RHF | sonnet | `68d4574` |
| 5 | Fields алхам нүүлгэх + legacy устгал | sonnet (дизайн=main opus) | `99b60aa` |

### Шинээр үүссэн / гол өөрчлөлт
- **`components/WizardStepGate.tsx`** (шинэ) — gate-registry context: `useRegisterWizardGate(async () => boolean)`, `useWizardStepGateRunner()`. Идэвхтэй алхам валидатораа бүртгэж, WizardShell "Үргэлжлүүлэх"-д дуудна.
- **`hooks/useRhfReduxBridge.ts`** (шинэ) — `form.watch` subscription → 1000ms debounce → Redux dispatch. Render-д watch-ийн утга ашиглахгүй (re-render үүсгэхгүй).
- **`schema/index.ts`** — `makeContentSchema`, `makeSubmitSchema`, `makeParticipantsSchema` factory-ууд (createType-aware; `lib/participants`/`lib/fields` helper-уудыг `superRefine`-д reuse).
- **4 step component** RHF болсон; **WizardShell.handleNext** gate-only.
- **Fields алхам**: canvas Redux хэвээр, gate нь `checkRequiredFields` → `fields.slice.setFieldErrors`; 3 компонент `selectFieldErrors` Redux selector уншина.
- **Устгасан**: `lib/validation-steps.ts`, `components/wizard-errors.ts` (бүх алдаа одоо RHF `formState.errors` эсвэл `fields.slice.errors`).

### Дизайны гол шийдлүүд
- **Алхам бүр тусдаа route → unmount**: RHF per-step `useForm`, `defaultValues`←Redux selector (survive); алхам хооронд Redux+localStorage+IndexedDB хадгална.
- **ParticipantStep async гишүүн шалгалт**: sync Zod resolver-д багтахгүй тул `asyncCheckRef` + `reportAsyncCheck`; gate = `trigger() && asyncOk`. `form.trigger()` гараар тавьсан алдааг арилгадаг RHF pitfall-ийг gate доторх resync-ээр шийдсэн.
- **SubmitStep**: terminal алхам, gate биш — `handleSubmit(onValid→AlertDialog, onInvalid→scroll+toast)`.
- **FieldSettingsPanel paramName**: controlled үлдээсэн (sanitize display-д тусах ёстой).

### Гүйцэтгэлийн инцидент (алхам 3)
Эхний opus agent connection алдаагаар тасарч, retry-тэй давхцаж race үүссэн (хоёулаа ижил файл бичсэн). 2 дахийг зогсоож, дискэн дээр коэрент үлдсэн дизайн (ParticipantStep + schema)-ийг хадгалаад, дутуу sub-component wiring-ийг main loop дээр өөрөө дуусгав. **Сургамж**: ижил файлд хүрэх subagent-уудыг параллель явуулахгүй (эсвэл worktree isolation).

### Үлдсэн (хэрэглэгч ажиллуулна)
- `npm run test:e2e` (Playwright) — wizard урсгал унахгүйг батлах.
- Гар **draft parity**: бөглөөд (1s) refresh → утга сэргэх; алхам хооронд back/next → хадгалагдах.
- `feat/contract-create-rhf` → review/merge.
