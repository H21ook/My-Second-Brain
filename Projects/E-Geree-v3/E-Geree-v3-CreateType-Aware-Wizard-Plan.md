---
title: "Plan: createType-aware wizard behavior"
type: project
status: draft
created: 2026-06-30
updated: 2026-07-01
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/plans/createtype-aware-wizard.md"
---

# Plan: createType-aware wizard behavior

**Огноо:** 2026-06-29  
**Салбар:** dev-khishigee  
**Төлөв:** ✅ Дууссан (2026-06-29)

---

## Зорилго

Wizard бүх `createType`-д яг адилхан ажилладаг. Дараах ялгааг нэмэх:

- **Participant 1 lock** — гэрээнд locked, загварт org locked / ажилтан солигддог
- **Validation relaxation** — загварт participant username + field value шаардахгүй
- **Submit strategy** — TEMPLATE/LINKED_TEMPLATE endpoint залгах
- **Payload** — `templateId`/`linkedTemplateId`/`contractEventId` зөв нэмэх

---

## Flows ба createType mapping

| Үйлдэл                | createType       | Flags            | P1 lock                       | Validation | Endpoint              |
| --------------------- | ---------------- | ---------------- | ----------------------------- | ---------- | --------------------- |
| Гэрээ илгээх          | PRIVATE_CONTRACT | —                | Бүрэн locked                  | Strict     | createContractRequest |
| Загвар үүсгэх         | TEMPLATE         | isCreateTemplate | Org locked, ажилтан солигддог | Relaxed    | createTemplate        |
| Загвар засах          | TEMPLATE         | isEditTemplate   | Org locked, ажилтан солигддог | Relaxed    | updateTemplate        |
| Холбоос загвар үүсгэх | LINKED_TEMPLATE  | isCreateTemplate | Org locked, ажилтан солигддог | Relaxed    | createLinkedTemplate  |
| Холбоос загвар засах  | LINKED_TEMPLATE  | isEditTemplate   | —                             | Relaxed    | updateLinkedTemplate  |
| Загвар ашиглах        | PRIVATE_CONTRACT | useTemplate=true | Бүрэн locked                  | Strict     | createContractRequest |
| Дахин илгээх          | RESEND           | —                | Бүрэн locked                  | Strict     | reSendEditContract    |

---

## Хэрэгжүүлэлтийн жагсаалт

### ✅ Дууссан

- [x] **`lib/create-type-helpers.ts`** (шинэ) — `isTemplateFlow(ct)` helper
- [x] **`api/queries.ts`** — `submitStrategies`-д TEMPLATE + LINKED_TEMPLATE нэмсэн
- [x] **`lib/payload.ts`** — `linkedTemplateId` + `contractEventId` conditional нэмсэн
- [x] **`lib/validation-steps.ts`** — `validateParticipant` + `validateFields` template-д relaxed. ⚠️ **Superseded (2026-06-30):** энэ файл RHF шилжилтээр устгагдсан — логик одоо `schema/index.ts` → `makeParticipantsSchema`/Zod дотор (`isTemplate`-aware). Дэлгэрэнгүй: [[E-Geree-v3-Contract-Create-RHF-Plan]].
- [x] **`components/steps/participant/SectionBasic.tsx`** — `disableLegalEntity` prop нэмсэн
- [x] **`components/steps/participant/SectionContact.tsx`** — `locked` + `orgLocked` props нэмсэн
- [x] **`components/steps/participant/ParticipantCard.tsx`** — `selectMeta` уншиж lock props дамжуулсан
- [x] **`components/steps/ParticipantStep.tsx`** — template горимд "Add recipient" товч нуусан
- [x] **`components/steps/SubmitStep.tsx`** — button label + field validation skip
- [x] **`i18n/languages-data/mn.json`** — `createTemplate` + `saveTemplate` keys нэмсэн

### ⏳ Үлдсэн — Wizard behavior

- [x] **`i18n/languages-data/en.json`** — `createTemplate` + `saveTemplate` нэмсэн
- [x] **`i18n/languages-data/cn.json`** — мөн адил
- [x] **`i18n/languages-data/kr.json`** — мөн адил
- [x] **TypeScript build шалгасан** — эрх гүй
- [x] **Vault reference update** — `04-Contract-Create-Feature.md` шинэчилсэн

---

## 2. Sidebar — "Баримт бичиг үүсгэх" entry point

### Зорилго

With-sidebar layout-д баримт бичиг үүсгэх entry point байхгүй. Sidebar-д профайлын доор "Баримт бичиг үүсгэх" товч нэмж, дарахад 4 төрлийн card харагдаж, сонгосноор тохирох `createType`-тэйгээр wizard-д ордог байхаар хэрэгжүүлнэ.

### UI дизайн

```
Sidebar
 ├─ SidebarHeader (TeamSwitcher / profile)
 ├─ [NEW] "Баримт бичиг үүсгэх" товч  ← профайлын доор
 └─ SidebarContent (NavMain, NavProjects…)
```

Товч дарахад **Dialog** нээгдэж, 2×2 card grid:

| Card | createType | Flags |
|---|---|---|
| Хувийн гэрээ | PRIVATE_CONTRACT | — |
| Загвар | TEMPLATE | isCreateTemplate: true |
| Холбоост загвар | LINKED_TEMPLATE | isCreateTemplate: true |
| Нээлттэй баримт бичиг | PUBLIC_CONTRACT | — |

Card сонгоход:
1. `clearDraft()` — localStorage draft цэвэрлэнэ
2. `dispatch(resetContractCreate())` — Redux state reset
3. `dispatch(setMeta({ createType, isCreateTemplate }))` — flow тохируулна
4. `router.push("/contract/create")` — wizard-д орно

### Хэрэгжүүлэлт

- [x] **`src/components/layout/CreateDocumentDialog.tsx`** (шинэ) — Dialog + 4 card
- [x] **`src/components/layout/app-sidebar.tsx`** — CreateDocumentDialog нэмсэн
- [x] **i18n** — `sidebar.privateContract` key (4 файл)

---

## Гол дизайн шийдвэрүүд

**`isTemplateFlow` хаана ашиглагдсан** (⚠️ жагсаалт бичигдсэн үеийн snapshot; RHF шилжилтийн дараа `validation-steps.ts` → `schema/index.ts` болсон):
```
lib/validation-steps.ts       → validateParticipant, validateFields   [DELETED, 2026-06-30 → schema/index.ts]
lib/payload.ts                → linkedTemplateId/contractEventId conditional
participant/ParticipantCard.tsx → senderLocked, senderOrgLocked
steps/ParticipantStep.tsx     → canAdd button
steps/SubmitStep.tsx          → handleTrySubmit, button label
```

**Submit endpoint сонголт — payload-аас derived:**
```typescript
// payload.templateId байвал → updateTemplate, үгүй → createTemplate
[CreateType.TEMPLATE]: (payload) =>
  payload.templateId ? updateTemplate(payload) : createTemplate(payload)
```

**Participant 1 lock logic (`ParticipantCard`):**
```typescript
const senderLocked    = isSender && !isTemplate  // гэрээ: бүрэн locked
const senderOrgLocked = isSender && isTemplate   // загвар: org locked, ажилтан free
```

**`SectionContact` lock тооцоолол:**
```typescript
const fullyLocked   = locked || (orgLocked && !isOrg)  // иргэн илгээгч загварт = locked
const orgInfoLocked = orgLocked && isOrg               // org илгээгч загварт = regNum locked only
```

---

## 2026-07-01 үргэлжлэл — Fields/Submit алхмын parity

Дээрх план "Дууссан" гэж тэмдэглэсэн байсан ч **Fields алхам загварт validation-гүй** байсан нь илэрсэн (`FieldsStep`-ийн gate `isTemplateFlow`-д шууд `true` буцаадаг байлаа → Continue шалгуургүй дарагддаг). Мөн Submit алхмын UI/copy/route бүгд гэрээний хувилбар л байсан. Дараах засварууд:

- [x] **`FieldsStep.tsx`** — template bypass хассан; `checkRequiredFields` одоо загварт ч гэрээтэй адил ажиллана (`lib/fields.ts`).
- [x] **`payload.ts`** — `contractType`: `TEMPLATE` → `SENT` (өмнө нь `content.config.contractType`-ээс авдаг байсан).
- [x] **`StepHeader.tsx`** — wizard гарчиг `createType`-с хамаарна (`wizardTitle*` i18n keys, 4 хувилбар).
- [x] **`SubmitStep.tsx`** — Сэндэр fill sidebar: header үлдэж, дотор нь загварт `templateNoPrefill` тайлбар (form-ын оронд). Баталгаажуулах Dialog copy (`confirmCreateTemplate*`) + action товч `tActions("yes")` бүх тохиолдолд. Амжилтын toast (`createTemplateSuccess`/`saveTemplateSuccess`) + redirect (`/documents/template` vs `/documents/sent`) createType-с хамаарна.
- [x] **`src/app/[locale]/.../documents/template/page.tsx`** (шинэ) — placeholder route, redirect-ийн зорилтот хуудас амжилттай resolve хийхийн тулд.
- [x] **`CreateDocumentDialog.tsx`** — карт дараалал: Хувийн гэрээ → Загвар → Нээлттэй баримт → Холбоост загвар (хамгийн доор). Доод 2 (Нээлттэй баримт, Холбоост загвар) `@todo re-enable` тэмдэглэлтэй түр disabled.

Branch: `feat/profile-dropdown-recent` (профайл feature-тэй хамт нэг branch дээр — тусдаа биш).
