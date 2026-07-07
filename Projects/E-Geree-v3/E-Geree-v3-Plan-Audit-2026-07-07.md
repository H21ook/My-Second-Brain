---
title: Plan аудит — дутуу зүйлсийн нэгтгэл
created: 2026-07-07
updated: 2026-07-07
tags: [egeree/plan, egeree/audit]
status: done
---

# Plan аудит — 2026-07-07

> [!note] Түр аудитын тэмдэглэл — §7 дараалал биелж, сургамж нь guide/reference-д distill хийгдсэний дараа устгаж болно.

> [!note] Энэ бол аудитын нэгтгэл тэмдэглэл. 14 plan-ыг тус бүр (1) шинжлэх → (2) adversarial батлах → (3) тэмдэглэх гэсэн 3 үе шаттайгаар код (`e-geree-v3`, branch `dev-khishigee`) болон worklog-уудтай тулгаж шалгав. Plan бүрд "Аудит 2026-07-07" callout нэмэгдсэн — дэлгэрэнгүй нотолгоо тэнд байгаа.

[[E-Geree-v3-Home]]

## 1. Ерөнхий дүр зураг

| Plan | Төлөв | Гүйцэтгэл | Дутуу |
|---|---|---|---|
| E-Geree-v3-Landing-Design-Solution (устгасан 2026-07-07 → [[E-Geree-v3-Landing]]) | ✅ done | 31/31 | — |
| E-Geree-v3-Landing-Phase0-2-Implementation-Plan (устгасан 2026-07-07 → [[E-Geree-v3-Landing]]) | ✅ done | 43/43 | — |
| E-Geree-v3-Landing-Phase3-Implementation-Plan (устгасан 2026-07-07 → [[E-Geree-v3-Landing]]) | ✅ done | 14/14 | — |
| E-Geree-v3-Landing-Phase4-5-Implementation-Plan (устгасан 2026-07-07 → [[E-Geree-v3-Landing]]) | ✅ done | 18/18 | — |
| E-Geree-v3-Landing-Redesign-Migration-Plan (устгасан 2026-07-07 → [[E-Geree-v3-Landing]]) | ✅ done | 20/20 | — |
| E-Geree-v3-Contract-Detail-Page-Phase2a-Plan (устгасан → [[E-Geree-v3-Contract-Detail]]) | ✅ done | 6/6 | 1 ⚪ |
| E-Geree-v3-Contract-Create-RHF-Plan (устгасан → [[E-Geree-v3-Contract-Create-Feature]]) | ✅ done | 12/14 | 2 ⚪ |
| E-Geree-v3-Contract-Detail-UX-Refinement-Plan (устгасан → [[E-Geree-v3-Contract-Detail]]) | ✅ done | 17/18 | 1 ⚪ |
| [[E-Geree-v3-AppSidebar-Profile-Dropdown-Plan]] | 🟨 mostly-done | 9/10 | 1 ⚪ |
| [[E-Geree-v3-Contract-Detail-Drawer-Phase1-Plan]] | 🟨 mostly-done | 10/12 | 3 ⚪ |
| [[E-Geree-v3-Contract-Detail-Page-Phase2c-Plan]] | 🟨 mostly-done | 9/11 | 1 🟡, 1 ⚪ |
| [[E-Geree-v3-Contract-Detail-Page-Phase2d-Plan]] | 🟨 mostly-done | 13/18 | 3 🟡, 2 ⚪ |
| [[E-Geree-v3-CreateType-Aware-Wizard-Plan]] | 🟨 mostly-done | 21/24 | 1 🟡, 1 ⚪ |
| [[E-Geree-v3-Profile-Migration-Plan]] | 🟨 mostly-done | 12/23 | **3 🔴**, 4 🟡, 2 ⚪ |

Нийт баталгаажсан дутуу зүйл: **26** (3 🔴 high, 9 🟡 medium, 14 ⚪ low). Landing-ийн 5 plan бүгд бүрэн — vault дүрмээр distill хийгээд устгаж болно.

> [!info] Аудитад ороогүй: [[E-Geree-v3-Landing-Phase6-Visual-Polish-Plan]] — 2026-07-07 18:16-д (аудит явж байх үед) үүссэн шинэ plan, `status: ready`, хараахан хэрэгжээгүй тул хамрагдаагүй. Залруулга (2026-07-07): эхний шалгалтад YAML эвдэрхий мэт харагдсан нь RTK output filter-ийн artifact байсан — raw-byte шалгалтаар frontmatter бүрэн хүчинтэй.

## 2. 🔴 High — хамгийн түрүүнд

Бүгд [[E-Geree-v3-Profile-Migration-Plan]]-д:

1. **Phase G3 огт эхлээгүй** — `organization/dashboard` + `organization/subscription` route байхгүй, `nav.ts:85,103` хоёулаа `enabled: false`, G3 commit алга. Pre-flight (recharts 2.15.4, icon assets) бэлэн.
2. **G3-г блоклож буй шийдвэр** — payment-modal reuse (hoist to `shared/` vs duplicate) гараагүй; `ff09d0b` commit modal-ыг documents дотор self-contained үлдээж "Deferred: subscription/template payments" гэж шууд тэмдэглэсэн.
3. **§17 Manual smoke gate 0/8** — B2 email merge, C security зэрэг auth-critical урсгалуудын live тест нэг ч хийгдээгүй (зөвхөн B1 SMS хэсэгчлэн).

## 3. 🟡 Medium

- **Live/browser verification хойшлогдсон** — [[E-Geree-v3-Contract-Detail-Page-Phase2d-Plan]]: GSIGN (DAN push утас), TRIDUM (`ws://127.0.0.1:59001` desktop клиент), Payment (QPay sandbox) гурвуулаа static+build confidence-т л хүрсэн; [[E-Geree-v3-Contract-Detail-Page-Phase2c-Plan]]: товчны enable/disable + 3 reason action + delete урсгал browser-т шалгаагүй (Worklog 2026-07-03: "Not verified in-browser").
- **cn/kr i18n gap** — [[E-Geree-v3-CreateType-Aware-Wizard-Plan]]: 8 key (wizardTitleTemplate, templateNoPrefill, confirmCreateTemplate* г.м.) mn/en-д байгаа ч cn.json/kr.json-д алга.
- **Profile §16 нээлттэй шийдвэрүүд** — subscription attach point-ууд (SubscriptionPlanCard `disabled` хэвээр), OrgVerificationHeader (v3-д 0 файл), ProfileNav mobile fallback (`w-56` fixed, breakpoint 0).
- **§18 deploy/infra** — PAYMENT_URL, PAYMENT_URL_V2, DIGITAL_SIGNATURE_URL зэрэг env-үүд repo-ийн ямар ч CI/Dockerfile-д байхгүй, out-of-band баталгаагүй; шинэ BFF surface-ийн security review бүртгэлгүй.

## 4. ⚪ Low (товчоор)

- **E2E тест** — `tests/e2e/critical-flows.spec.ts`-д wizard flow-ийн тест огт байхгүй; `test-results` mtime 2026-06-30 = түүнээс хойш `npm run test:e2e` ажиллаагүй (RHF, 2c, 2d гурвуулаа энэ өртэй).
- **Manual QA checklist-үүд pending** — AppSidebar dropdown (recent list, F1, dialog search), UX-Refinement (fill-guard, sticky header, dock 7 кейс), RHF draft parity (refresh → сэргэлт).
- **Drawer Phase1 үлдэгдэл** — QR код модал огт хэрэгжээгүй; "Устгах"/"Нууц үг солих" dropdown item-ууд comment-out; ReceivedContent inline useQuery → api модуль руу зөөх (optional).
- **Phase2a** — SecuredDetailContent буруу accessCode үед алдааны мессеж/toast харуулдаггүй (чимээгүй gate дахин гарна).
- **CreateDocumentDialog** desc keys (createDocumentDesc г.м.) cn/kr-д дутуу.
- **Profile tracked deferral-ууд** — Phase D signature, user/settings, api-connection, DAN linking г.м. — бүгд §16 ledger-т эзэнтэй, мартагдаагүй.

## 5. Давтагдсан хэв маяг (системийн ажиглалт)

1. **Frontmatter `status:` хуучирсан — 8 plan.** AppSidebar ("approved, not implemented"!), Drawer Phase1, 2a, 2c, 2d, CreateType, Landing-Redesign (draft), Design-Solution (approved) — бүгд бодит байдалд хэрэгжсэн. Кодтой таарч байгаа нь зөвхөн RHF, UX-Refinement, Phase3, Phase4-5, Profile.
2. **Browser/live verification системтэйгээр хойшлогддог** — бараг бүх plan-ы нийтлэг дутагдал. Build/tsc/lint gate сайн ажиллаж байгаа ч live gate байхгүй.
3. **Worklog convention зөрчил** — Landing-ийн бүх ажил (~30 commit) болон Profile-ийн 07-07 gap-audit өдрийн worklog-uudad огт бичигдээгүй; plan-ийн дотоод worklog хэсэгт л бий. Worklog-2026-07-07 зөвхөн GSIGN + label-sidebar-ыг хамарсан.
4. **Шинэ илэрсэн bug ledger-т ороогүй** — EmployeeSelector-ийн `/company-employee` буруу endpoint (Worklog 07-07-д "not fixed, separate scope") Profile plan-ийн §16-д бүртгэгдээгүй.

## 6. Хэрэглэгчийн TODO-той уялдаа

`Untitled.md`-д бичсэн зүйлс энэ аудиттай давхцаж байна:
- "Create template үйлдлийг шалгаж эцэслэнэ" → CreateType wizard-ийн cn/kr i18n + template flow live шалгалт (§3)
- "Received list-г шалгаж дутуу зүйлсийг гүйцээнэ" → Drawer Phase1-ийн comment-out item-ууд + ReceivedContent refactor (§4)
- "Contract detail" → 2c/2d-ийн live verification өр (§3)

## 7. Санал болгох дараалал

1. **G3 pre-flight шийдвэрүүд** (payment-modal reuse, OrgVerificationHeader) → дараа нь G3 — цорын ганц 🔴 блок.
2. **§17 smoke gate** — auth-critical B2/C-ээс эхлэх.
3. **Quick win:** cn/kr i18n 13 key нөхөх (8 wizard + 5 dialog desc).
4. Wizard e2e тест нэмж `npm run test:e2e`-г сэргээх.
5. 8 plan-ы frontmatter status шинэчлэх; ✅ болсон Landing 5 + RHF + 2a + UX-Refinement plan-уудыг distill → устгах ([[E-Geree-v3-Note-Keeping-Guide]] дүрмээр).
6. EmployeeSelector endpoint bug-ийг §16 ledger-т нэмэх.

## Documentation Impact

- [x] [[E-Geree-v3-Note-Keeping-Guide]] — §5-ын process сургамжууд (status sync, live-verify gate) нэмэгдсэн
- [x] Дууссан plan-ууд устгагдмагц энэ note-ийн линкүүд шинэчлэгдэнэ — 8 plan устгаж, §1-ийн линкүүдийг de-link хийсэн (2026-07-07)
