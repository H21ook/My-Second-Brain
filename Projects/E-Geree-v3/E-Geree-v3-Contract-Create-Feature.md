---
title: "04 — Гэрээ үүсгэх feature"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/04-Contract-Create-Feature.md"
---

# Contract Create — гол feature

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-Routing]] · дараах: [[E-Geree-v3-State-Management]] · төлөвлөгөө: [[E-Geree-v3-RHF-Migration-Plan]]

`src/features/contract-create/` — бие даасан, өөрийн бүх давхаргатай модуль.

## Дотоод бүтэц
| Хавтас | Үүрэг |
|---|---|
| `components/steps/` | Wizard алхмууд: `participant/`, `content/`, `fields/` |
| `store/` | Feature-scope Redux slice-ууд. [[E-Geree-v3-State-Management]] |
| `hooks/` | `useWizardNav`, `useFieldKeyboardShortcuts`, `useParticipantsSeed`, `useHydrated` |
| `lib/` | `payload.ts`, `participants.ts`, `fields.ts`, `participant-colors.ts`, `pdf-url.ts`, `validation-steps.ts` |
| `api/` | `client.ts`, `queries.ts`, `upload.ts` (talбар зураг/PDF байршуулах) |
| `config/` | `steps.ts` — алхмын дараалал, href тооцоо |
| `schema/` | Zod validation |
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

## createType-aware wizard behavior

`meta.createType` wizard-ийн бүх гол шийдвэрийг тодорхойлно. `lib/create-type-helpers.ts` дахь `isTemplateFlow(ct)` helper (`TEMPLATE | LINKED_TEMPLATE` → `true`).

| createType | Endpoint | P1 lock | Validation |
|---|---|---|---|
| `PRIVATE_CONTRACT` | `createContractRequest` | Бүрэн locked | Strict |
| `TEMPLATE` (create) | `createTemplate` | Org locked | Relaxed |
| `TEMPLATE` (edit) | `updateTemplate` | Org locked | Relaxed |
| `LINKED_TEMPLATE` (create) | `createLinkedTemplate` | Org locked | Relaxed |
| `LINKED_TEMPLATE` (edit) | `updateLinkedTemplate` | — | Relaxed |
| `PUBLIC_CONTRACT` | `createPublicContract` | Бүрэн locked | Strict |

**Relaxed validation** (`isTemplateFlow = true`): participant username + field value шаардахгүй.

**Participant 1 lock (`ParticipantCard`)**:
```ts
senderLocked    = isSender && !isTemplate  // гэрээ: бүрэн locked
senderOrgLocked = isSender && isTemplate   // загвар: org locked, ажилтан free
```

**Submit endpoint** (`api/queries.ts` → `submitStrategies`): `payload.templateId` байвал `updateTemplate`, үгүй бол `createTemplate`.

**Entry point**: `src/components/layout/CreateDocumentDialog.tsx` — sidebar-аас 4 card dialog. Сонгоход `clearDraft → resetContractCreate → setMeta({ createType, isCreateTemplate }) → /contract/create`.

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
| **Илгээх амжилттай** (`SubmitStep.handleSubmit`) | — `clearDraft`/`resetContractCreate` **commented out** | ❌ GAP (refresh хийхэд хуучин draft үлдэнэ) |
| **Unmount** (`WizardBootstrap` teardown) | Эцсийн state-г localStorage руу flush | ⚠️ blob цэвэрлэгддэггүй |
| **Хуудаснаас гарах** | — | ❌ Цэвэрлэлтгүй |

## Анхаарах зүйлс / TODO
- **Submit cleanup идэвхжүүлэх** — `SubmitStep.handleSubmit` дотор `clearDraft` + `resetContractCreate`
  (+ амжилттай хуудас руу redirect) commented байгааг бодит mutation залгахдаа сэргээх.
- **Сервер upload-ын дараа blob цэвэрлэх** — local PDF серверт орсны дараа IndexedDB blob үлддэг.
- **Хуучирсан draft-ийн expiry алга** — орхисон draft localStorage/IndexedDB-д хуримтлагдаж болзошгүй.
- **Form control rearchitect** — Redux-аас react-hook-form руу шилжүүлэх төлөвлөгөө: [[E-Geree-v3-RHF-Migration-Plan]].
