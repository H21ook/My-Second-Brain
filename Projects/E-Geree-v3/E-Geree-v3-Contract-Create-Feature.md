---
title: "04 — Гэрээ үүсгэх feature"
type: project
status: draft
created: 2026-06-30
updated: 2026-07-01
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/04-Contract-Create-Feature.md"
---

# Contract Create — гол feature

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-Routing]] · дараах: [[E-Geree-v3-State-Management]] · form control: [[E-Geree-v3-Contract-Create-RHF-Plan]] (done, 2026-06-30)

`src/features/contract-create/` — бие даасан, өөрийн бүх давхаргатай модуль.

## Дотоод бүтэц
| Хавтас | Үүрэг |
|---|---|
| `components/steps/` | Wizard алхмууд: `participant/`, `content/`, `fields/` |
| `store/` | Feature-scope Redux slice-ууд. [[E-Geree-v3-State-Management]] |
| `hooks/` | `useWizardNav`, `useFieldKeyboardShortcuts`, `useParticipantsSeed`, `useHydrated`, `useRhfReduxBridge` (RHF↔Redux гүүр) |
| `lib/` | `payload.ts`, `participants.ts`, `fields.ts` (incl. `checkRequiredFields`), `participant-colors.ts`, `pdf-url.ts`, `create-type-helpers.ts`, `labels.ts` |
| `api/` | `client.ts`, `queries.ts`, `upload.ts` (talбар зураг/PDF байршуулах) |
| `config/` | `steps.ts` — алхмын дараалал, href тооцоо |
| `schema/` | Zod validation (`makeContentSchema`/`makeSubmitSchema`/`makeParticipantsSchema` — runtime-д ашиглагддаг, createType-aware) |
| `types/` | Feature-н TS төрлүүд |
| `index.ts` | Нийтийн barrel — step компонентуудыг export-лоно (`WizardBootstrap`, `WizardShell`, `ContentStep`, `FieldsStep`, `ParticipantStep`, `SubmitStep`) |

App доторх 5 route нь гүн зам биш `@/features/contract-create` barrel-ээс import-лоно.
Root store reducer-ыг шууд `@/features/contract-create/store`-ээс авна.

## Wizard урсгал
```
Content → Participant → Fields → Submit
```
(Дараалал нь `config/steps.ts` дахь `StepKey` массиваар тодорхойлогдоно.)
1. **Content** — гэрээний агуулга (PDF / TEXT), гарчиг, дуусах хугацаа (expire day)
2. **Participant** — гэрээний оролцогчид (хувь хүн / хуулийн этгээд), өнгөөр ялгана
   (`participant-colors.ts`)
3. **Fields** — PDF дээр талбар (гарын үсэг, текст г.м.) байрлуулна → [[E-Geree-v3-PDF-Viewer]]
4. **Submit** — `payload.ts` нь бүх slice-ийг нэгтгэж BFF-ээр илгээнэ → [[E-Geree-v3-Networking-BFF]]

## Form control (react-hook-form)

Форм input-ууд бүгд Redux dispatch-аар биш **RHF**-аар удирддаг (2026-06-30, [[E-Geree-v3-Contract-Create-RHF-Plan]]). Redux = алхам хоорондын source of truth (persist/IndexedDB), RHF = алхам доторх (per-field validate/error). Гүүр:

- **`components/WizardStepGate.tsx`** — gate-registry context: идэвхтэй алхам `useRegisterWizardGate(async () => boolean)`-аар бүртгэнэ; `WizardShell.handleNext` → `useWizardStepGateRunner()`.
- **`hooks/useRhfReduxBridge.ts`** — `form.watch` subscription → 1000ms debounce → Redux dispatch (render-д ашиглахгүй тул нэмэлт re-render үгүй).
- **Fields canvas** нь эксепшн — Redux хэвээр (drag/drop, undo); зөвхөн `FieldSettingsPanel`-ийн input RHF.
- Устгагдсан: `lib/validation-steps.ts`, `components/wizard-errors.ts` (алдаа одоо RHF `formState.errors` эсвэл `fields.slice.errors`-оор).

## createType-aware wizard behavior

`meta.createType` wizard-ийн бүх гол шийдвэрийг тодорхойлно. `lib/create-type-helpers.ts` дахь `isTemplateFlow(ct)` helper (`TEMPLATE | LINKED_TEMPLATE` → `true`).

| createType | Endpoint (`api/queries.ts` → `submitContract`) | P1 lock | Participant validation | Fields validation |
|---|---|---|---|---|
| `PRIVATE_CONTRACT` | `createPrivateContract` | Бүрэн locked | Strict | Strict |
| `TEMPLATE` (create/edit, `payload.templateId` derived) | `createTemplate` / `updateTemplate` | Org locked | Relaxed | Strict (2026-07-01-ээс хойш) |
| `LINKED_TEMPLATE` (create/edit, `payload.linkedTemplateId` derived) | `createLinkedTemplate` / `updateLinkedTemplate` | Org locked | Relaxed | Strict (2026-07-01-ээс хойш) |
| `RESEND` | `resendPrivateContract` | Бүрэн locked | Strict | Strict |
| `PUBLIC_CONTRACT` | ⚠️ `submitContract`-д хэрэгжээгүй (`default` throws) | Бүрэн locked | Strict | Strict |

**Relaxed participant validation** (`isTemplateFlow = true`, `schema/index.ts` → `makeParticipantsSchema`): зөвхөн participant1 (илгээгч)-ийн identity заавал; бусад оролцогч заавал биш, SIGN config-ийн verification бүгдэд шалгагдана.

**Fields validation (2026-07-01 засвар):** өмнө нь `FieldsStep`-ийн gate загварт (`isTemplateFlow`) `checkRequiredFields`-ийг бүрэн алгасдаг байсан тул Continue шалгуургүй дарагддаг байлаа. Одоо загвар ч гэрээтэй ЯГ АДИЛХАН `lib/fields.ts` → `checkRequiredFields`-ээр шалгагдана (FORM_FILLER талбар, SIGNATURE/STAMP verification, хоосон SELECT сонголт).

**Participant 1 lock (`ParticipantCard`)**:
```ts
senderLocked    = isSender && !isTemplate  // гэрээ: бүрэн locked
senderOrgLocked = isSender && isTemplate   // загвар: org locked, ажилтан free
```

**Submit endpoint** (`api/queries.ts` → `submitContract`): `payload.templateId`/`linkedTemplateId` байвал update, үгүй бол create.

**Entry point**: `src/components/layout/CreateDocumentDialog.tsx` — sidebar-аас 4 card dialog (list layout, icon tint, тайлбартай). Дараалал: Хувийн гэрээ → Загвар → Нээлттэй баримт → Холбоост загвар (хамгийн доор). ⚠️ 2026-07-01: доод 2 сонголт (Нээлттэй баримт, Холбоост загвар) `@todo re-enable` тэмдэглэлтэй түр disabled. Сонгоход `clearDraft → resetContractCreate → setMeta({ createType, isCreateTemplate }) → /contract/create`.

**Wizard title** (`StepHeader.tsx`, 2026-07-01): `createType`-с хамааран өөрчлөгдөнө — `wizardTitle` (гэрээ) / `wizardTitleTemplate` / `wizardTitleLinkedTemplate` / `wizardTitlePublic`.

**Submit step (`SubmitStep.tsx`) — загвар vs гэрээ ялгаа (2026-07-01):**
- Баруун sidebar-ын "Оролцогч 1-ийн бөглөх" header харагдана, гэхдээ дотор нь бөглөх form-ын оронд `templateNoPrefill` тайлбар текст ("Загвар тул талбаруудыг урьдчилж бөглөх боломжгүй").
- Баталгаажуулах Dialog: загварт `confirmCreateTemplateTitle`/`Desc`, гэрээнд хуучин `confirmSendTitle`/`Desc`. Action товч бүх тохиолдолд `tActions("yes")`.
- `payload.ts`: `contractType` — `TEMPLATE` одоо `PRIVATE_CONTRACT`-тэй адил `SENT`-рүү mapping-тэй (өмнө нь `content.config.contractType`-ээс авдаг байсан).
- Амжилттай toast: загварт `createTemplateSuccess`/`saveTemplateSuccess` (`isEditTemplate`-аас хамааран), гэрээнд `sendSuccess`.
- Redirect: загвар → `/documents/template` (шинэ placeholder route), гэрээ → `/documents/sent`.

## Гол төрлүүд
- `ParticipantConfig`, `Participant` — оролцогчийн тохиргоо
- `ContractField` — PDF талбар (төрөл, координат, оролцогчид хамаарах)
- `LegalEntityType` — хувь хүн эсвэл хуулийн этгээд

## Төлөв ба хамаарал (State dependencies)
Feature store нь 4 slice-ийг нэгтгэнэ (`store/index.ts`), root action `resetContractCreate`
(бүх slice-ийг initial болгох) ба `hydrateContractCreate` (draft-аас бүтэн орлуулах). Дэлгэрэнгүй
[[E-Geree-v3-State-Management]].

- **`meta.contentType` ↔ `content`** — PDF үед `relatedPdfUrl` (сервер) / `relatedPdfLocalUrl`
  (blob preview); TEXT үед `content` (HTML). Аль нэг идэвхтэй.
- **`participantKey` нь `participants` ↔ `fields`-ийг холбоно** — талбар бүр оролцогчид хамаарна.
  ⚠️ Оролцогчийг устгахад түүний талбарууд автоматаар устдаггүй (store-д cascade-delete алга).
- **`content.config.participantsConfig[]`** — оролцогч тус бүрийн permission/verification/secure,
  `participantKey`-ээр индекслэгдэнэ.
- **`fields`** — `groupId` (duplicate/group property sync), `order` (оролцогч тус бүрд 1..n reindex).

## Дата хадгалалт (Persistence)
- **`store/persist.ts`** — localStorage key `CONTRACT_CREATE_DRAFT`, throttle **500ms**-аар store-д
  subscribe. `loadDraft` нь сэргээхдээ `relatedPdfLocalUrl`-г цэвэрлэдэг (blob URL session-only).
- **`store/pdf-storage.ts`** — PDF blob нь IndexedDB-д (`egeree-contract-create`, store `pdf`, key
  **`"current"`** — нэг идэвхтэй PDF). `savePdfBlob` / `loadPdfBlob` / `clearPdfBlob`.
- **Boot дараалал** (`WizardBootstrap`): `hydrateDraft` (localStorage → state, IndexedDB-ээс blob
  сэргээж шинэ blob URL үүсгэх) → `setupContractCreatePersistence` (subscribe). `useHydrated` нь
  hydrate дуустал spinner үзүүлнэ (SSR-safe).

## Дата цэвэрлэх flow (Cleanup)
| Үйлдэл | Юу цэвэрлэгдэх | Төлөв |
|---|---|---|
| **Цуцлах** (`WizardShell.handleCancel`) | `resetContractCreate` + `clearDraft` (localStorage + IndexedDB blob) | ✅ Бүрэн |
| **Content → дараагийн алхам** (`handleNext`) | Сонгоогүй төрлийн контент (TEXT эсвэл PDF) + TEXT үед blob | ⚠️ Хэсэгчилсэн (localStorage draft хэвээр) |
| **Илгээх амжилттай** (`SubmitStep.handleConfirmedSubmit`) | `clearDraft` + `resetContractCreate` + redirect | ✅ Идэвхтэй (өмнө нь commented байсан, засагдсан) |
| **Unmount** (`WizardBootstrap` teardown) | Эцсийн state-г localStorage руу flush | ⚠️ blob цэвэрлэгддэггүй |
| **Хуудаснаас гарах** | — | ❌ Цэвэрлэлтгүй |

## Анхаарах зүйлс / TODO
- **Сервер upload-ын дараа blob цэвэрлэх** — local PDF серверт орсны дараа IndexedDB blob үлддэг.
- **Хуучирсан draft-ийн expiry алга** — орхисон draft localStorage/IndexedDB-д хуримтлагдаж болзошгүй.
- **`/documents/template`** — одоогоор placeholder page (`<div>TemplatePage</div>`), жагсаалт хараахан хэрэгжээгүй.
- **`PUBLIC_CONTRACT` submit strategy хэрэгжээгүй** — `api/queries.ts` → `submitContract`-ийн `default` case throw хийдэг (Нээлттэй баримт card CreateDocumentDialog-д одоогоор disabled — LINKED_TEMPLATE-ийн endpoint хэрэгжсэн хэдий ч мөн disabled).
