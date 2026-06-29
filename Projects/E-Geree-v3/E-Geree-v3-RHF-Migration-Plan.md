---
title: "RHF шилжүүлэх төлөвлөгөө (Contract-Create)"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/plans/RHF-Migration-Plan.md"
---

# react-hook-form шилжүүлэх төлөвлөгөө — Contract-Create

> [!note] Энэ бол **хэрэгжүүлэх төлөвлөгөө** — хэрэгжсэний дараа устгаж болно. Тогтмол reference биш.

[[E-Geree-v3-Contract-Create-Feature]] · холбоотой: [[E-Geree-v3-State-Management]]

Гэрээ үүсгэх wizard-ийн form/error/value control одоо **Redux dispatch-аар (keystroke бүрт)** явдаг. Validation нь "Үргэлжлүүлэх" дарахад `lib/validation-steps.ts`-ийн гар бичсэн validator-аар ажилладаг. Багийн RHF туршлагыг ашиглаж form-уудыг **react-hook-form + Zod** руу шилжүүлэх.

---

## 1. Зорилго ба хамрах хүрээ (Hybrid)

**RHF болгох (form inputs):**
- ContentStep — title/content/subject/description + config switch-ууд
- ParticipantStep — оролцогчдын жагсаалт + тус бүрийн config (permission/verification/secure)
- SubmitStep — илгээгчийн бөглөх талбарууд
- FieldSettingsPanel — сонгосон талбарын props (нэр, required, x/y, options г.м.)

**Redux хэвээр (RHF-д тохирохгүй):**
- FieldsStep canvas — талбар drag/drop байршуулах, `undo` history, `groupId` sync, `order` reindex (`store/fields.slice.ts`)
- Draft persist — `store/persist.ts` (localStorage throttle 500ms), `store/pdf-storage.ts` (IndexedDB blob)
- Cross-step мета (`meta` slice), wizard navigation (`useWizardNav`)

**Яагаад hybrid:** FieldsStep нь form биш — canvas. Persist давхарга Redux store-д subscribe хийдэг тул form-уудыг бүхэлд нь RHF руу зөөвөл draft хадгалалтыг дахин зохиомжлох шаардлагатай болж эрсдэл нэмэгдэнэ.

---

## 2. Архитектур

- Алхам тус бүрд нэг `useForm` (дэд хэсэгт хуваагдсан алхамд `FormProvider` + `useFormContext`).
- RHF нь тухайн алхмын input-уудын **source of truth** болно; `defaultValues` нь Redux selector-оос ачаална.
- Лавлах загвар: `src/components/custom/login/login-form.tsx` (`useForm` + `zodResolver` + RTK Query mutation) — энэ паттернийг алхам бүрд хуулбарлана.

---

## 3. Redux-тэй гүүр (persistence bridge)

Одоогийн draft/IndexedDB flow-г эвдэхгүйн тулд **RHF → Redux sync** хэвээр үлдээнэ:

- `defaultValues` ← Redux selector (`selectContent`, `selectParticipants`, ...).
- `watch` (эсвэл `onBlur`) → throttled `dispatch` → Redux → одоогийн `persist.ts` subscriber → localStorage. (RHF өөрөө keystroke бүрт dispatch хийхгүй; throttle хийнэ.)
- PDF blob нь Redux/IndexedDB-д хэвээр (RHF хөндөхгүй).

> Сонголт Б (зөвлөхгүй): draft-ийг бүхэлд нь RHF руу зөөж persist-ийг RHF дээр дахин бичих — илүү цэвэр гэвч том өөрчлөлт, эрсдэлтэй.

---

## 4. Zod схемийг бодит болгох

`schema/index.ts`-ийн skeletal схемүүд одоо runtime-д ашиглагддаггүй. Эдгээрийг бодит дүрэм болгож өргөтгөж, `zodResolver`-аар холбоно. **Логик давхардуулахгүй** — одоо байгаа helper-уудыг `superRefine` дотроос дуудна:

- `EMAIL_REG`, `REGISTRY_REG`, `isValidUsername` ← `lib/participants.ts`
- `isVerificationValid` ← `lib/participants.ts` (SIGN → SIGNATURE/DIGITAL_SIGNATURE)
- `checkRequiredFields` ← `lib/fields.ts` (FORM_FILLER, SIGNATURE/STAMP, SELECT options)

Ингэснээр `validation-steps.ts`-ийн дүрэм Zod руу нэг эх сурвалжаас шилжинэ.

---

## 5. Алхам тус бүрийн зураглал

| Алхам | RHF хэрэгсэл | Тэмдэглэл |
|---|---|---|
| **ContentStep** | `useForm<ContentSchema>` | title/content/subject/description + switch-ууд. PDF upload нь Redux/IndexedDB хэвээр. |
| **ParticipantStep** | `useForm` + `useFieldArray` | оролцогчдын массив; nested config; гишүүн шалгах **async** урсгал (`useCompanyEmployeeCheck`)-ийг `setValue`/`setError`-оор хослуулна. |
| **SubmitStep** | `useForm` | илгээгчийн бөглөх талбарууд; required-ийг RHF-аар (одоогийн `handleTrySubmit`-ийг орлоно); алдаатай талбар руу scroll хэвээр. |
| **FieldSettingsPanel** | жижиг `useForm` | сонгосон талбарын props; submit дээр `updateField`/`updateGroupFields` dispatch. |
| **FieldsStep canvas** | — | Redux хэвээр; зөвхөн settings input RHF. |

---

## 6. Validation интеграци

- `WizardErrorsProvider` / `useFieldError` / `useWizardErrors`-г RHF `formState.errors`-оор орлоно.
- WizardShell-ийн "Үргэлжлүүлэх" → тухайн алхмын `form.trigger()` / `handleSubmit`; зөв бол `goNext`.
- Алдааны одоогийн key-конвенц (`${participantKey}:field`, `select:${id}`)-ийг RHF талбарын зам (`participants.0.username`, `fields.<id>.options`) руу буулгана. Auto-select (алдаатай оролцогч/талбар сонгох) логикийг `formState.errors`-оос уншиж хадгална.

---

## 7. Шилжүүлэх дараалал (incremental)

Алхам бүр тусдаа PR/commit, бусдыг эвдэхгүйгээр ship хийнэ:

1. **ContentStep** (хамгийн энгийн — pilot, паттерн тогтооно)
2. **SubmitStep** (бөглөх талбарууд, required)
3. **ParticipantStep** (хамгийн хүнд — field array + async гишүүн шалгалт)
4. **FieldSettingsPanel**

Эхний алхамд Zod + persistence-bridge + WizardShell интеграцийг хамт суурилуулна.

---

## 8. Эрсдэл ба tradeoff

- Шилжилтийн үед зарим алхам RHF, зарим нь Redux — **давхар source-of-truth**. Алхам бүрийг бүрэн дуусгаж merge хийх.
- `useFieldArray` key тогтворжилт (`participantKey`-г `keyName` болгож ашиглах).
- Draft sync timing — RHF `watch` → throttled dispatch хоорондын зөрүү (одоогийн 500ms throttle-тэй уялдуулах).
- RHF↔Redux давхцлаас render давхардал гарахаас сэргийлэх (`watch` хязгаарлах, subscribe-ийг шаардлагатай талбарт).

---

## 9. Rollback

Per-step additive тул алхам бүрийг буцааж болно. Энэ note нь хэрэгжсэний дараа устгагдах working doc.

---

## Шалгах (verification)

- Алхам бүрийн дараа: `npx tsc --noEmit` + `npm run lint` цэвэр.
- Одоогийн Playwright E2E (`npm run test:e2e`) wizard урсгалд унахгүй байх.
- Draft persist гар тест: бөглөөд refresh хийхэд утга сэргэх (localStorage + IndexedDB blob).
- Validation parity: одоогийн `validation-steps.ts` дүрэм бүр Zod дээр давтагдсан эсэх.
