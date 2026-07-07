---
title: "04 — Гэрээ үүсгэх feature"
type: project
status: active
created: 2026-06-30
updated: 2026-07-07
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/04-Contract-Create-Feature.md"
---

# Contract Create — гол feature

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-Routing]] · дараах: [[E-Geree-v3-State-Management]] · form control: RHF шилжилт (2026-06-30 дууссан, plan устгагдсан 2026-07-07)

`src/features/contract-create/` — бие даасан, өөрийн бүх давхаргатай модуль.

## Дотоод бүтэц
| Хавтас | Үүрэг |
|---|---|
| `components/steps/` | Wizard алхмууд: `participant/`, `content/`, `fields/`, `submit/` |
| `store/` | Feature-scope Redux slice-ууд. [[E-Geree-v3-State-Management]] |
| `hooks/` | `useWizardNav`, `useFieldKeyboardShortcuts`, `useParticipantsSeed`, `useRhfReduxBridge` (RHF↔Redux гүүр) |
| `lib/` | `payload.ts`, `participants.ts`, `fields.ts` (incl. `checkRequiredFields`), `create-type-helpers.ts` |
| `api/` | `client.ts`, `queries.ts` |
| `config/` | `steps.ts` — алхмын дараалал, href тооцоо |
| `schema/` | Zod validation (`makeContentSchema`/`makeSubmitSchema`/`makeParticipantsSchema` — runtime-д ашиглагддаг, createType-aware) |
| `types/` | Feature-н TS төрлүүд |
| `index.ts` | Нийтийн barrel — step компонентууд + gate provider-ыг export-лоно (`WizardBootstrap`, `WizardShell`, `WizardStepGateProvider`, `ContentStep`, `FieldsStep`, `ParticipantStep`, `SubmitStep`) |

### Shared руу гарсан модулиуд (2026-07-03 … 2026-07-06)

Contract Detail (Phase 2d) болон profile-ийн ажлын явцад дараах модулиуд cross-feature хэрэглээтэй болж feature дотроос **shared байрлал руу** нүүсэн (feature-ийн дотоод import-ууд шинэ зам руу шинэчлэгдсэн):

| Хуучин (contract-create дотор) | Шинэ байрлал | Commit / огноо |
|---|---|---|
| `lib/pdf-url.ts` (`resolvePdfUrl`) | `src/shared/pdf-viewer/utils/pdf-url.ts` | `e928bc5`, 2026-07-03 |
| `lib/participant-colors.ts` | `src/lib/participant-colors.ts` | `43db7df`, 2026-07-03 |
| `lib/labels.ts` (+test) | `src/lib/labels.ts` | `b4ece39`, 2026-07-03 |
| `api/upload.ts` | `src/lib/upload.ts` | `b4ece39`, 2026-07-03 |
| `SenderFillSection` | `src/shared/field-fill/SenderFillSection.tsx` | `b4ece39`, 2026-07-03 |
| `hooks/useHydrated.ts` | `src/hooks/useHydrated.ts` (`@/hooks/useHydrated`) | `85c73d2`, 2026-07-06 |

App доторх 5 route нь гүн зам биш `@/features/contract-create` barrel-ээс import-лоно.
Root store reducer-ыг шууд `@/features/contract-create/store`-ээс авна.

## Wizard урсгал
```
Content → Participant → Fields → Submit
```
(Дараалал нь `config/steps.ts` дахь `StepKey` массиваар тодорхойлогдоно.)
1. **Content** — гэрээний агуулга (PDF / TEXT), гарчиг, дуусах хугацаа (expire day)
2. **Participant** — гэрээний оролцогчид (хувь хүн / хуулийн этгээд), өнгөөр ялгана
   (`src/lib/participant-colors.ts` — 2026-07-03-аас shared)
3. **Fields** — PDF дээр талбар (гарын үсэг, текст г.м.) байрлуулна → [[E-Geree-v3-PDF-Viewer]]
4. **Submit** — `payload.ts` нь бүх slice-ийг нэгтгэж BFF-ээр илгээнэ → [[E-Geree-v3-Networking-BFF]]

## Form control (react-hook-form)

Форм input-ууд бүгд Redux dispatch-аар биш **RHF**-аар удирддаг (RHF шилжилт 2026-06-30; plan нь distill хийгдээд устгагдсан). Redux = алхам хоорондын source of truth (persist/IndexedDB), RHF = алхам доторх (per-field validate/error). Загварын ерөнхий тайлбар: [[React-Hook-Form-Redux-Wizard-Pattern]]. Гүүр:

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
  сэргээж шинэ blob URL үүсгэх) → `setupContractCreatePersistence` (subscribe). `useHydrated`
  (2026-07-06-аас shared `src/hooks/useHydrated.ts`, `WizardShell.tsx:49`-д хэрэглэгддэг) нь
  hydrate дуустал spinner үзүүлнэ (SSR-safe).

## Дата цэвэрлэх flow (Cleanup)
| Үйлдэл | Юу цэвэрлэгдэх | Төлөв |
|---|---|---|
| **Цуцлах** (`WizardShell.handleCancel`) | `resetContractCreate` + `clearDraft` (localStorage + IndexedDB blob) | ✅ Бүрэн |
| **Content → дараагийн алхам** (`handleNext`) | Сонгоогүй төрлийн контент (TEXT эсвэл PDF) + TEXT үед blob | ⚠️ Хэсэгчилсэн (localStorage draft хэвээр) |
| **Илгээх амжилттай** (`SubmitStep.handleConfirmedSubmit`) | `clearDraft` + `resetContractCreate` + redirect | ✅ Идэвхтэй (өмнө нь commented байсан, засагдсан) |
| **Unmount** (`WizardBootstrap` teardown) | Эцсийн state-г localStorage руу flush | ⚠️ blob цэвэрлэгддэггүй |
| **Хуудаснаас гарах** | — | ❌ Цэвэрлэлтгүй |

## Мэдэгдэж буй хязгаарлалт

Дараах нь дизайны бодит төлөв (2026-07-07-нд `dev-khishigee` кодтой тулгаж баталгаажуулсан):

- **IndexedDB blob нь сервер upload-аас хамааралгүй амьдардаг** — `clearPdfBlob` зөвхөн PDF-ээ устгах (`PdfUploadField.tsx:62`), Цуцлах (`persist.ts:36` — `clearDraft` дотор) болон амжилттай илгээлтийн дараа (`SubmitStep.tsx:186`) дуудагдана; local PDF серверт орсны дараа blob өөрөө цэвэрлэгддэггүй.
- **Draft-д expiry/TTL механизм байхгүй** — `store/persist.ts` timestamp хадгалдаггүй тул орхисон draft хэрэглэгч Цуцлах дартал эсвэл амжилттай илгээтэл localStorage/IndexedDB-д хугацаагүй үлдэнэ.
- **`/documents/template` нь placeholder** — `src/app/[locale]/(protected)/(with-sidebar)/documents/template/page.tsx` = `<div>TemplatePage</div>`; загвар илгээсний дараах redirect энэ хуудас руу очдог ч жагсаалт хэрэгжээгүй (2026-07-07 байдлаар).
- **`PUBLIC_CONTRACT` submit strategy хэрэгжээгүй** — `api/queries.ts:63`-ийн `submitContract` `default` case `Promise.reject` буцаана. `CreateDocumentDialog.tsx:20-21`-д "Нээлттэй баримт" (PUBLIC_CONTRACT) ба "Холбоост загвар" (LINKED_TEMPLATE) card хоёулаа `disabled: true` — LINKED_TEMPLATE-ийн endpoint-ууд хэрэгжсэн ч UI entry нь хаалттай хэвээр.

Актив ажил: cn/kr i18n 8 key-ийн дутагдал болон template flow-ийн live баталгаажуулалт [[E-Geree-v3-CreateType-Aware-Wizard-Plan]] (in-progress)-д бүртгэлтэй.
