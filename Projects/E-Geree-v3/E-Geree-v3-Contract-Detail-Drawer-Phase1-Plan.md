---
title: "Plan: Contract Detail Drawer — Phase 1 (dropdown actions + rep modal + navigation)"
type: project
status: draft
created: 2026-07-02
updated: 2026-07-02
tags:
  - project
  - project/e-geree-v3
  - plan
---

# Plan: Contract Detail Drawer — Phase 1

**Огноо:** 2026-07-02  
**Салбар:** dev-khishigee  
**Төлөв:** 📝 Ноорог (батлагдаагүй)  
**Холбоотой:** [[E-Geree-v3-Contract-Create-Feature]] · [[E-Geree-v3-Networking-BFF]] · [[E-Geree-v3-State-Management]] · [[E-Geree-v3-Home]]

---

## Context (яагаад)

`e-geree-v3`-т `ContractDetailDrawer` (`src/features/documents/components/ContractDetailDrawer.tsx`) нь UI хувьд бэлэн ч **бүх үйлдэл stub** (хоосон `onClick`). Мөн detail рүү орох урсгал дутуу: `goToDetail` нь `relatedParticipantKeyList.length > 1` үед зүгээр `return` хийж юу ч гарахгүй. `e-geree-v2`-т энэ тохиолдолд **"Төлөөлөх оролцогч тал сонгох"** модал гарч, хэрэглэгч аль оролцогчоор нэвтрэхээ сонгодог.

Энэ төлөвлөгөө v2-ийн логикийг судалж **Phase 1**-ийг хэрэгжүүлнэ:
- Drawer доод dropdown үйлдлүүдийг v2-ийн бодит жагсаалтад тааруулж, статусаар нөхцөлдүүлэн ажиллуулах.
- Олон-төлөөлөгч сонгох модал (RelatedParticipantChooseModal) порт хийх.
- Detail рүү зөв чиглүүлэх navigation-г засах.

### Хамрах хүрээ (тохирсон)
- **Phase 1 (энэ):** drawer dropdown + rep modal + navigation.
- **Гарын үсэг / 2FA / тоон гарын үсэг:** хойшлуулсан (stub).
- **Dropdown жагсаалт:** v2-ийн бодит жагсаалтад тааруулна.

### Хойшлуулсан (Phase 2 — энэ төлөвлөгөөнд ОРОХГҮЙ)
Бүтэн detail page (`/documents/detail/[...slug]`), rightSidebar (SecondSidebar — оролцогч/нэмэлт файл/түүх accordion), detail доторх `ActionButtons` (sign/review/return/resend/cancel/accept-decline cancellation/pay), гарын үсэг зурах + 2FA/OTP (`SecurityVerificationModal`) + тоон гарын үсэг (`DigitalSignature`), PDF viewer холболт.

---

## v3 дээр бэлэн, дахин ашиглах хэсгүүд

| Хэрэгцээ | Бэлэн эх сурвалж | Замд |
|---|---|---|
| React Query + apiClient + `/backend` BFF загвар | `contractCreateApi` + hooks | `src/features/contract-create/api/{client,queries}.ts` |
| Browser HTTP client (envelope unwrap) | `apiClient` (axios, `baseURL:"/backend"`) | `src/core/http/api-client.ts` |
| Модал primitive | `Dialog*` | `src/components/ui/dialog.tsx` |
| Хайлттай сонголтын модал загвар | `AllProfilesDialog` (`CommandDialog`) | `src/shared/ui/components/all-profiles-dialog.tsx` |
| Устгах баталгаажуулалт | `alert-dialog.tsx` | `src/components/ui/alert-dialog.tsx` |
| Dropdown | `DropdownMenu*` (footer-т аль хэдийн ашиглагдаж буй) | `src/components/ui/dropdown-menu.tsx` |
| Оролцогч №, мөнгө формат | `getParticipantNumber`, `moneyFormatter` | `src/shared/lib/custom-utils.ts` |
| i18n navigation | `useRouter` | `@/i18n/navigation` |
| Toast | `sonner` (dep) | — |

Профайл сэлгэх урсгалын загвар (Phase 2 accordion switch-д): `src/hooks/useProfileSwitch.ts`, `src/actions/auth-actions.ts` (`switchProfileAction`).

---

## Ажлын зүйлс (Phase 1)

### 1. Documents API модуль — `src/features/documents/api/`
`contract-create/api`-ийн загварыг яг мөрдөнө: `client.ts` (endpoint-ууд apiClient дээр) + `queries.ts` (`useMutation`/`useQuery` + key factory).

Phase 1-д хэрэгтэй методууд:
- `deleteContract(id)` → POST `contract-request/remove/{id}` (body: null)
- `changeAccessCode(payload)` → POST v2 `contract-request/reset-access-code`
- `getLabelList()` → GET v2 `contract-label`; `setLabel` → POST v2 `contract-action/set-label`; `unsetLabel` → POST v2 `contract-action/unset-label`
- Link/relation endpoint-уудыг v2 `contract.js` L144-180-аас гаргаж нэмэх (5.c)

`ReceivedContent.tsx`-ийн inline `useQuery`-г мөн энэ модульд `useReceivedList` болгож зөөж болно (заавал биш).

> **Backend хувилбар баталгаажуулах:** v2-т `BACKEND_URL` / `BACKEND_URL_V2` / `BACKEND_URL_V3` гэсэн 3 backend. v3 BFF нь `/public-api/v1` ба `/public-api/v2` route-той. Received-list нь v2 дээр тул remove/label-ийг эхэлж v2-оор оролдоно; backend-тэй тулгаж баталгаажуулах. Phase 1-ийн бүх дуудлага **GET/POST** тул v3 BFF-ийн `okNoop` (v2 дээрх PUT/PATCH/DELETE) асуудал үүсгэхгүй. Дэлгэрэнгүй: [[E-Geree-v3-Networking-BFF]].

### 2. Олон-төлөөлөгч сонгох модал — `src/features/documents/components/RelatedParticipantChooseModal.tsx`
v2 `src/components/custom/related-participant-choose-modal/index.jsx`-ийн `"contract"` салааг порт хийнэ.
- `Dialog` дээр суурилна. Гарчиг: "Төлөөлөх оролцогч тал сонгох".
- `contract.relatedParticipantKeyList`-ийг давтаж, key бүрийг `contract.actionList` (`participantKey`)-тай тааруулна.
- Мөр бүр: "Оролцогч #{n}" + `action.nickname` + `action.secure` үед `Lock` icon.
- `onSelect(action)` → `router.push('/documents/detail/{contract.id}/{action.id}')` → модал хаах.
- Props: `contract: ContractListItem | null`, `open`, `onOpenChange`.

### 3. `goToDetail` + модалын state — `ContractDetailDrawer.tsx`
Одоогийн `goToDetail` (L113-134)-ийг v2-ийн `handleRouteContractDetail` (`second-sidebar/index.jsx` L124-153) нөхцөлд тааруулна:

```
length === 0                          → юу ч хийхгүй (эрхгүй)
length === 1  ЭСВЭЛ  status COMPLETED → push /documents/detail/{id}/{action.id}
                                         (action = actionList.find(participantKey === related[0]))
бусад (length > 1 && !COMPLETED)      → setChooseModalOpen(true)  (модал нээх)
```

Модалын `useState`-ийг drawer дотор нэмж `<RelatedParticipantChooseModal .../>`-г рендэрлэнэ. "Дэлгэрэнгүй харах" товч (L312-319) хэвээр `goToDetail`-г дуудна.

### 4. Footer dropdown-г v2-ийн бодит жагсаалтад тааруулах — `ContractDetailDrawer.tsx` (L104-111, L338-351)
`availableActions`-ийг статусаар нөхцөлдсөн, v2 `renderActionButtons` (`second-sidebar/index.jsx` L195-477)-тэй ижил item set болгоно. `'sign'`-г **хасна** (v2-т sign нь зөвхөн detail page дээр).

| Item | Нөхцөл | Үйлдэл (Phase 1) |
|---|---|---|
| PDF татах | `status COMPLETED` | Client anchor → `signedPdfUrl \|\| generatedPdfUrl`. **Бүрэн.** |
| Гэрчилгээ татах | `COMPLETED && certificatePdfUrl` | Client anchor → `certificatePdfUrl`. **Бүрэн.** |
| Устгах | destructive | Баталгаажуулах модал → `deleteContract(id)` → refetch. **Бүрэн (5.a).** |
| Холбоос холбох | v2 нөхцөл | `LinkContractModal` нээх (5.c — sub-modal). |
| Хавтас нэмэх/зөөх/хасах | v2 нөхцөл | Label модал нээх (5.d — sub-modal). |
| Нууц үг солих | `action.secure` | `PasswordChangeModal` нээх (5.e — sub-modal). |
| QR код | `type==="public" && APPROVED` | QR модал (public list байхгүй бол алгасаж болно). |

> **Phasing зөвлөмж:** Dropdown item + нөхцөлийг бүрэн тодорхойлно. Handler холболтыг 2 давхаргад хуваана:
> - **1a (нугас):** PDF/гэрчилгээ татах + Устгах (баталгаажуулалттай) + оролцогч сонгох модал + navigation. Шинэ modal-гүй бүрэн ажиллана.
> - **1b (sub-modal бүрд тус модал + endpoint):** Холбоос, Хавтас(label), Нууц үг, QR — тус бүр тусад нь.

### 5. Дэмжих компонентууд
- **a. DeleteConfirmDialog** — `alert-dialog.tsx` дээр. Устгахын өмнө баталгаажуулах → `useDeleteContract` mutation → амжилтад `toast` (sonner) + received-list `invalidateQueries`/`refetch` + drawer хаах.
- **b. Toast** — `sonner`; v2-ийн `useToast`-ийн орлуулга.
- **c. LinkContractModal** (1b) — v2 `LinkContractModal` + `getContractRelationList`/`linkContract`/`unlinkContract` (v2 `contract.js` L144-180) порт.
- **d. LabelModal** (1b) — v2 `useLabel` урсгал: `getLabelList` + `setLabel`/`unsetLabel`.
- **e. PasswordChangeModal** (1b) — v2 `ContractPasswordChangeModal` + `resetContractAccessCode` (`contract-request/reset-access-code`).

---

## Endpoint лавлагаа (v2-ээс баталгаажсан)

| Үйлдэл | Method | Path (v2 дахь) | v2 backend const |
|---|---|---|---|
| Гэрээ устгах | POST | `/contract-request/remove/{id}` (body: null) | `BACKEND_URL` |
| Нууц үг (access-code) солих | POST | `/contract-request/reset-access-code` | `BACKEND_URL_V2` |
| Мэдэгдэл дахин илгээх | POST | `/contract-request/resend-notification` | `BACKEND_URL_V2` |
| Label жагсаалт | GET | `/contract-label` | `BACKEND_URL_V2` |
| Label оноох / хасах | POST | `/contract-action/set-label` · `/contract-action/unset-label` | `BACKEND_URL_V2` |
| PDF / гэрчилгээ татах | — | client anchor (`signedPdfUrl`/`generatedPdfUrl`/`certificatePdfUrl`) | — |

Phase 2 (detail page) лавлагаанд: `contract-action/approve`, `/review`, `/return`, `/send-cancel-request`, `/accept-cancellation`, `/decline-cancellation`, `contract-request/resend`.

---

## Verification (Phase 1)

1. **Build/lint/knip:** `npm run lint && npm run knip && npx tsc --noEmit` — шинэ файлууд unused export үлдээхгүй.
2. **Олон-төлөөлөгч урсгал:** `relatedParticipantKeyList.length > 1` && статус ≠ COMPLETED гэрээ дээр "Дэлгэрэнгүй харах" → модал гарч, оролцогч сонгоход `/documents/detail/{id}/{actionId}` рүү үсрэх. `length === 1` эсвэл COMPLETED үед модалгүйгээр шууд үсрэх.
3. **Татах:** COMPLETED гэрээн дээр PDF/гэрчилгээ item гарч, зөв URL татах. COMPLETED биш үед item нуугдана.
4. **Устгах:** dropdown → Устгах → баталгаажуулах модал → устгасны дараа жагсаалтаас алга + toast (E2E mock эсвэл dev backend).
5. **E2E:** `npm run test:e2e` (critical-flows эвдрээгүйг батлах); шаардвал drawer→модал урсгалд нэг тест нэмэх.
6. Гар аргаар `npm run dev`: received жагсаалтаас мөр дарж drawer нээх, дээрх урсгалуудыг гүйцэтгэх.

---

## Файлын товч жагсаалт

**Шинэ:**
- `src/features/documents/api/client.ts`, `src/features/documents/api/queries.ts`
- `src/features/documents/components/RelatedParticipantChooseModal.tsx`
- `src/features/documents/components/DeleteConfirmDialog.tsx`
- (1b) `LinkContractModal.tsx`, `LabelModal.tsx`, `PasswordChangeModal.tsx`

**Засах:**
- `src/features/documents/components/ContractDetailDrawer.tsx` (goToDetail, модал state, footer dropdown)
- `src/types/enums.ts` (`ContractStatusEnum` байхгүй бол нэмэх)
- (заавал биш) `src/features/documents/components/ReceivedContent.tsx` (inline useQuery → api модуль)

---

## v2 эх сурвалжийн лавлагаа (порт хийхэд)

- Rep-choose модал: `e-geree-v2/src/components/custom/related-participant-choose-modal/index.jsx`
- goToDetail нөхцөл: `e-geree-v2/.../documents/second-sidebar/index.jsx` L124-153
- Footer үйлдлүүд: тэрхүү `second-sidebar/index.jsx` `renderActionButtons` L195-477
- Endpoint-ууд: `e-geree-v2/src/lib/actions/{contract,contractV2,label}.js`

---

## Гүйцэтгэлийн дараалал, модел, токен хэмнэлт

### Дараалал (алхам алхмаар)

| #    | Алхам                                                                                | Файл                                                     | Хамаарал | Санал болгох модел                    |
| ---- | ------------------------------------------------------------------------------------ | -------------------------------------------------------- | -------- | ------------------------------------- |
| 0    | `ContractStatusEnum` нэмэх (байхгүй бол)                                             | `src/types/enums.ts`                                     | —        | Haiku 4.5 (trivial)                   |
| 1a-1 | Documents api модуль — эхлээд зөвхөн `deleteContract` + key factory                  | `documents/api/{client,queries}.ts`                      | 0        | Sonnet 5                              |
| 1a-2 | `RelatedParticipantChooseModal` порт                                                 | `documents/components/RelatedParticipantChooseModal.tsx` | —        | Sonnet 5                              |
| 1a-3 | `goToDetail` нөхцөл засах + модал state холбох                                       | `ContractDetailDrawer.tsx`                               | 1a-2     | **Opus 4.8** (нөхцөлийн логик нарийн) |
| 1a-4 | Footer dropdown бүтэц + PDF/гэрчилгээ татах + `DeleteConfirmDialog` + delete холболт | `ContractDetailDrawer.tsx`, `DeleteConfirmDialog.tsx`    | 1a-1     | Sonnet 5                              |
| ✅    | **1a verify + commit** (доорх verification 1–4)                                      | —                                                        | —        | —                                     |
| 1b-1 | `LinkContractModal` + link/relation endpoint                                         | шинэ + api                                               | —        | Sonnet 5                              |
| 1b-2 | `LabelModal` + label endpoint-ууд                                                    | шинэ + api                                               | —        | Sonnet 5                              |
| 1b-3 | `PasswordChangeModal` + reset-access-code                                            | шинэ + api                                               | —        | Sonnet 5                              |
| 1b-4 | QR модал (public урсгал байвал)                                                      | шинэ                                                     | —        | Sonnet 5                              |
| ✅    | **1b verify + commit**                                                               | —                                                        | —        | —                                     |
| —    | Phase 2 (detail page + sidebar + sign/2FA)                                           | тусдаа төлөвлөгөө/чат                                    | —        | Opus (архитектур)                     |

Ерөнхий дүрэм: **1a бүрэн (verify+commit) → дараа нь 1b**. Алхам бүр = жижиг diff = 1 commit.

### Effort түвшин (`/effort` — токен хэмнэх өөр гол хөшүүрэг)
Effort-ийг ажлын нарийн ширийнд тааруул. Өндөр effort = илүү reasoning token; механик ажилд дэмий.

| Алхам төрөл | Алхмууд | Effort |
|---|---|---|
| Механик / нэг файлын порт, тодорхой markup | 0, 1a-2, 1b-1..4 | **low–medium** |
| Шинэ api модуль, mutation + query wiring | 1a-1, 1a-4 | **medium** |
| Нөхцөлт салаа логик, олон файл холбоо | 1a-3 (goToDetail) | **high** |
| Phase 2 архитектур / бүтэн detail page | Phase 2 | **xhigh** |

Дүрэм: default **medium**; нарийн логикт л **high**+ болго; порт хийхэд **low**-руу буулгаж токен хэмнэ. Модел ↔ effort хамт: Haiku/low, Sonnet/medium, Opus/high–xhigh. Тохирох модел effort-г subagent тохируулж ашиглана.

### Чат стратеги (токен хэмнэх гол)
- **1a — одоогийн чатаа үргэлжлүүл.** Контекст аль хэдийн ачаалагдсан, дахин explore хэрэггүй.
- **1b болон Phase 2 — фаза бүрд ШИНЭ чат.** Seed: энэ plan note + доорх target файлуудыг л унших. Хуучин explore контекст үүрэхгүй.
- Шинэ чат seed prompt загвар: *"Read this plan note: `<зам>`. Implement алхам 1b-1 зөвхөн. Read зөвхөн: `ContractDetailDrawer.tsx`, `documents/api/client.ts`, v2 `LinkContractModal`."*

### Токен-бага, чанар-өндөр ажлын горим
1. **Дахин explore хийхгүй** — plan note бол эх сурвалж; бүх зам/endpoint дотор нь бий.
2. **Зорилтот файл л унших** — алхмын хүснэгтэд заасан файлууд төдий. Бүтэн директор скан хийхгүй.
3. **graphify эхэлж** (repo дүрэм): `graphify query "..."` → дараа нь тодорхой мөр засах/унших.
4. **Тусгаарлагдсан 1–2 файлын механик порт** → `cavecrew-builder` subagent (гаралт шахагдсан → main контекст бага иддэг). Ж: модал markup порт.
5. **Алхам бүрийн дараа**: тухайн файлд `tsc --noEmit` + `eslint` → commit. Бүх зүйлийг эцэст нь бус.
6. `knip`-ийг 1a/1b төгсгөлд л ажиллуулна (unused export шалгах).
7. caveman + ponytail асаалттай хэвээр (богино diff, богино тайлбар).

### Одоогийн ачаалагдсан контекст (үргэлжлүүлбэл дахин уншихгүй)
`ContractDetailDrawer.tsx` (бүтэн), `contract-types.ts`, `contract-create/api/{client,queries}.ts`, v2-ийн rep-modal + second-sidebar + endpoint map — бүгд энэ чатад бий.
