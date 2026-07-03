---
title: "Plan: Contract Detail Page — UX Refinement (breadcrumb + action dock + rail tabs)"
type: project
status: done
created: 2026-07-03
updated: 2026-07-03
tags:
  - project
  - project/e-geree-v3
  - plan
  - ux
---

# Plan: Contract Detail — UX Refinement

**Огноо:** 2026-07-03
**Салбар:** dev-khishigee
**Төлөв:** ✅ Дууссан (5 commit)
**Холбоотой:** [[E-Geree-v3-Contract-Detail-Page-Phase2d-Plan]] · [[E-Geree-v3-PDF-Viewer]] · [[E-Geree-v3-Routing]] · [[E-Geree-v3-Contract-Detail-Page-Phase2a-Plan]]

Layout redesign (`9e6409b`)-ийн дараах **хэрэглэгчийн ойлгомж** засвар. Product-design-preflight → 3 шийдэл батлагдсан (3 commit) + хэрэглэгч гараар layout тааруулж, tab active-state bug олдож, fill-guard нэмсэн (4-р commit) + Бөглөх tab-ийн title/desc sticky болгосон (5-р commit).

---

## Context (яагаад)

Layout redesign 2-багана bounded rail + toolbar + FlowActionDock авчирсан. Гэвч 3 friction үлдсэн:

1. **Rail дахь 2 зорилго холилдсон.** `FieldFillPanel` (DO — бөглөх талбар) + `ContractSidebar` (READ — оролцогч/файл/түүх) нэг scroll хайрцагт дараалсан. Талбар олон бол READ хэсэг fold-оос доош унаж **харагдахгүй**.
2. **Breadcrumb түүхий.** `DynamicBreadcrumb` зөвхөн pathname-аас → `Баримт бичиг → detail(орчуулгагүй) → <id> → <id>`. Нэр/оролцогч алга.
3. **Primary шатлал бүрхэг.** `ActionButtons` олон inline secondary товч (Буцаах/Дахин илгээх/Төлбөр/зөвшөөрөх) toolbar-т шуугиан үүсгэнэ; primary 2 газар (dock + toolbar) тарсан.

---

## Батлагдсан шийдэл (preflight)

### 1. Rail — нөхцөлт Tab
- `fillActive` үед `Tabs`: **Бөглөх** (default, `FieldFillPanel`) | **Мэдээлэл** (`ContractSidebar`). Dock 2 tab-д pinned.
- `!fillActive` үед tab-гүй, `ContractSidebar` шууд (нэг зорилготой → tab chrome илүүц).
- `ponytail`: gate = `fillActive` (талбар 0 signer → "Бөглөх" tab хоосон; ховор кейс, dock-ийн Баталгаажуулах гол үйлдэл хэвээр).

### 2. Actions — rail-footer нэгдсэн dock (drawer footer загвар)
- `FlowActionDock` + `ActionButtons`-ийг **нэг dock** болгож нэгтгэнэ: `[ primary (flex-1) ] [ ⋯ ]`.
- **Primary resolver (яг нэг):** flow (`getFlowAction`: Баталгаажуулах/Хянах/Тоон гарын үсэг) → байхгүй бол fallback (Дахин илгээх → Хүсэлт зөвшөөрөх → Төлбөр төлөх).
- **`⋯` overflow:** үлдсэн бүгд — Буцаах, Засах, Хүсэлт татгалзах, Гэрээ цуцлах, Устгах (destructive → destructive өнгө).
- Mutation/modal/confirm-dialog wiring **хэвээр** — зөвхөн trigger layout нүүнэ.
- Toolbar → нэр + status badge + copy-ID л үлдэнэ.

### 3. Breadcrumb
`Баримт бичиг → Ирсэн/Илгээсэн → [гэрээний нэр] → Оролцогч N`
- `Ирсэн`/`Илгээсэн` = `actionData.sender` (sender → Илгээсэн `/documents/sent`, эс бол Ирсэн `/documents/received`).
- `[нэр]` = `data.title` (non-link, muted). `Оролцогч N` = `getParticipantNumber(actionData.participantKey)` (leaf, `aria-current`).
- **Boundary-safe:** `DynamicBreadcrumb` нь `src/components/layout` (app-component) → feature hook импортлож болохгүй ([[feature-boundaries-eslint]]). Тиймээс `src/shared/breadcrumb` context slot: detail page `useSetBreadcrumb([...])`, breadcrumb context байвал custom items, эс бол pathname fallback хэвээр. Дахин ашиглагдана.

---

## Commit задаргаа

### Commit 1: Breadcrumb slot
- **NEW** `src/shared/breadcrumb/breadcrumb-context.tsx` — `BreadcrumbProvider` + `useBreadcrumb` + `useSetBreadcrumb`.
- **NEW** `src/shared/breadcrumb/index.ts` — public API.
- `layout.tsx` (with-sidebar) — header + children-ийг `BreadcrumbProvider`-т ор.
- `dynamic-breadcrumb.tsx` — context items байвал custom branch, эс бол pathname fallback.
- `ContractDetail.tsx` — `useSetBreadcrumb(useMemo(...))` (data/actionId-аас нэр+sender+participantN).

### Commit 2: Action dock merge
- `ActionButtons.tsx` — dock layout болгож дахин бичих (`[primary][⋯]`, `getFlowAction` primary resolver, sticky footer). flow handler prop-ууд (onApprove/onReview/onDigitalSign) нэмэх (2d-2c placeholder).
- `FlowActionDock.tsx` — **устгах** (нэгдсэн).
- `ContractDetail.tsx` — toolbar-оос `ActionButtons` авах; rail footer-ийн `FlowActionDock`-г `ActionButtons`-оор солих.

### Commit 3: Rail tabs
- `ContractDetail.tsx` — rail body-г `fillActive` үед `Tabs` (Бөглөх/Мэдээлэл), эс бол `ContractSidebar` шууд. `Tabs` flex-col, `TabsContent` overflow-y-auto, dock footer-т pinned хэвээр.

---

## Verify (commit бүрт)
- `tsc --noEmit` 0 алдаа · `eslint` (boundaries + naming) 0 · `knip` dead-code 0 · `vitest` regression алга.
- Manual (auth session шаардлагатай, headless боломжгүй): fillActive tab солих, breadcrumb нэр/оролцогч, dock primary кейсүүд (approve/review/digital/resend/accept/pay/none), destructive overflow, `<lg` stack.

---

## Хэрэгжилт

Салбар `dev-khishigee`. Verify commit бүрт: **tsc 0 · eslint 0 (boundaries+naming) · knip шинэ dead-code алга · vitest 28/28**.

### ✅ Commit 1 — breadcrumb (`ddce598`)
- **NEW** `src/shared/breadcrumb/breadcrumb-context.tsx` — `BreadcrumbProvider` + `useBreadcrumb` + `useSetBreadcrumb` (unmount дээр null болгож цэвэрлэнэ). `.tsx` shared/-д naming-glob-т үл хамаарна; boundary: feature/app → shared зөвшөөрөгдсөн.
- **NEW** `src/shared/breadcrumb/index.ts` — public API.
- `(with-sidebar)/layout.tsx` — header+children-ийг `BreadcrumbProvider`-т орооov.
- `dynamic-breadcrumb.tsx` — context items байвал custom (i18n `Link` locale-prefix), эс бол pathname fallback хэвээр.
- `ContractDetail.tsx` — `useSetBreadcrumb(useMemo(...))`: `data.title` + `actionData.sender` (Ирсэн/Илгээсэн) + `getParticipantNumber` (Оролцогч N). Hook-ийг early-return-оос ӨМНӨ дуудсан (rules-of-hooks).

### ✅ Commit 2 — action dock (`c2c07e5`)
- `ActionButtons.tsx` — нэгдсэн dock `[primary][⋯]`. Primary resolver: `getFlowAction` → resend → accept → pay. Overflow: буцаах/засах/татгалзах/цуцлах/устгах (destructive `variant`). Mutation/modal/confirm wiring хэвээр. flow handler prop-ууд (onApprove/onReview/onDigitalSign, 2d-2c placeholder). `dockActive` prop устгав.
- `FlowActionDock.tsx` — **устгасан** (нэгдсэн).
- `ContractDetail.tsx` — toolbar identity-only; dock rail-footer-т `ActionButtons`.
- `activePayButton` одоо `getFlowAction(...) === null`-аар шалгана (өмнө `!isFieldFillActive`).

### ✅ Commit 3 — rail tabs (`097af94`)
- `ContractDetail.tsx` — `fillActive` үед `Tabs` (Бөглөх default / Мэдээлэл), TabsContent `flex-1 min-h-0 overflow-y-auto`; эс бол `ContractSidebar` шууд. Dock footer-т pinned. FieldFillPanel-ийн утга parent `fields` state-д тул tab unmount-д алдагдахгүй.

### ✅ Commit 4 — tab bug fix + layout polish + fill-guard (`4286b81`)
Хэрэглэгч гараар `ContractDetail.tsx` бүтцийг өөрчилсний дараах засвар (toolbar+viewer+rail → нэг bordered карт).

- **Bug (pre-existing, 4 tab consumer бүгдэд):** `src/components/ui/tabs.tsx`-ийн `TabsTrigger` bare `data-active:` selector ашигладаг байсан. Radix Tabs `data-state="active"` тавьдаг, ганц `data-active` attr биш → selector хэзээ ч тохирохгүй, идэвхтэй tab харагдахгүй байсан. Fix: бүх `data-active:` → `data-[state=active]:`.
- **Height chain:** гараар restructure хийхэд `ViewerPlaceholder`-ийн `lg:h-full`-д хэрэгтэй bounded flex ancestor алдагдсан. Bordered wrapper → `flex flex-1 flex-col lg:min-h-0`, дотоод grid мөр → `flex-1 lg:min-h-0`. Хэрэглэгчийн `gap-4` арилгасан шийдвэр болон `TabsContent`-ийн шинэ border/padding хэвээр — зөвхөн height classname засав.
- `FieldFillPanel.tsx` — `rounded-lg border p-3` wrapper устгасан (өөрийн tab pane-д илүүц).
- **Fill-guard:** `ContractDetail.tsx` rail `Tabs` одоо controlled (`railTab` state). `fields.some(...)`-аар тухайн оролцогчийн `FILLABLE_TYPES` талбар бүгд бөглөгдсөн эсэхийг шалгаад, `ActionButtons`-ийн `onApprove`-д дутуу бол `setRailTab("fill")` (шилжинэ, дарааллаа зогсооно). Бодит submit хэвээр 2d-2c placeholder.

### ✅ Commit 5 — Бөглөх tab sticky header (`58eb940`)
Олон талбартай үед signer scroll хийхэд гарчиг/заавар алдагдана — sticky болгов.

- `SenderFillSection.tsx` — шинэ `showDescription` prop (default `true`, SubmitStep хэвээр өөрчлөгдөөгүй), FieldFillPanel өөрийн толгойг харуулах үед `false`.
- `FieldFillPanel.tsx` — `sticky -top-4 -mx-4 -mt-4 px-4 pt-4` толгой (гарчиг+заавар). **Яагаад ийм тоо:** эцэг `TabsContent` өөрөө `p-4` (scroll container) тул түүний **дээд padding scroll-той хамт хэзээ ч арилдаггүй** (CSS зан) → sticky доор байнга тунгалаг зай үлдэж, арын мөр цухуйна. Тиймээс sticky блок `-mx-4 -mt-4`-ээр эцгийн padding-г цуцалж, `px-4 pt-4`-ээр дахин үзүүлээд, `-top-4`-ээр stick цэгээ тохируулна — анх ачаалахад padding хэвээр харагдана, scroll хийхэд sticky bg арын контентыг бүрэн бүрхэнэ.
- `ContractDetail.tsx` — `TabsContent`-ийн `p-4`-г буцаан шууд дээр нь (commit 4-т wrapper div-т шилжүүлсэн байсныг revert, учир нь хэрэглэгч padding харагдалтыг хадгалахыг хүссэн — зөвхөн bleed-through л засуулсан).

### Дараа (manual QA — auth session, headless боломжгүй)
fillActive tab солих (идэвхтэй tab харагдах эсэх) · fields дутуу үед Баталгаажуулах дарж Бөглөх tab руу шилжих эсэх · Бөглөх tab scroll хийхэд толгой sticky + арын мөр цухуйхгүй эсэх · breadcrumb нэр/оролцогч/Ирсэн-Илгээсэн · dock primary кейс бүр (approve/review/digital/resend/accept/pay/none) · destructive overflow · `<lg` stack · height/scroll (`lg:min-h-0` chain, PDF `lg:h-full`).
