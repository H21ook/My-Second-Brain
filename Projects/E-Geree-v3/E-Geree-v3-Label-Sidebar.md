---
title: "Label Sidebar — хавтасны мод (NavLabels)"
type: project
status: active
created: 2026-07-07
updated: 2026-07-07
tags:
  - project
  - project/e-geree-v3
---

# Label Sidebar — хавтасны мод (NavLabels)

[[E-Geree-v3-Home]] · холбоотой: [[E-Geree-v3-Networking-BFF]], [[E-Geree-v3-Routing]] · эх сурвалж: [[E-Geree-v3-Worklog-2026-07-07]]

Sidebar-ын хэрэглэгчийн хавтас (contract-label) систем — v2-ийн `MenuLabelList`-ийн v3 порт. shadcn demo `NavProjects`-ийг орлуулсан: `app-sidebar.tsx`-д `<NavLabels />` (`src/components/layout/app-sidebar.tsx:17,115`), `nav-projects.tsx` устгагдсан. Бүх зүйл 2026-07-07-нд live session-д (CITIZEN + HADES LLC ORGANIZATION) баталгаажсан.

## Гол файлууд

| Файл | Үүрэг |
|---|---|
| `src/components/layout/nav-labels.tsx` | Хавтасны мод, kebab цэс, dialog state, drop target |
| `src/components/layout/ShareLabelDialog.tsx` | Root хавтсыг баг/гишүүдэд хуваалцах (org) |
| `src/features/documents/components/{Create,Rename,Delete}LabelDialog.tsx` | CRUD dialog-ууд |
| `src/features/documents/api/client.ts:107-155` | Label endpoint-ууд + `LabelNode`/`TeamLabelEntry` төрлүүд |
| `src/features/documents/api/queries.ts` | `useLabelList` + мутацийн hook-ууд |
| `src/features/documents/lib/drag.ts` | Native DnD MIME төрөл + `contractActionIds()` |
| `src/features/documents/components/received-content/ReceivedToolbar.tsx` | Toolbar-ын хавтас chip-ууд |

## Дата бүтэц (`client.ts:126-155`)

- `LabelNode` = `{ contractLabel: { id, name, colorHex?, totalCount?, parentLabelId?, viewAll?, teamLabelList? }, children? }`. **Хаа сайгүй зөвхөн `contractLabel.id`** хэрэглэнэ (wrapper id биш — v2-ийн ID хоёрдмол байдлыг давтахгүй).
- `TeamLabelEntry` (`client.ts:142-146`) — хандах эрхтэй нэг мөр: `team` ЭСВЭЛ `employee` (аль нэг нь); `id` = teamLabelId бөгөөд хасалтад хэрэглэнэ.
- `GET .../v2/contract-label` хариу нь эдгээр optional талбарыг (colorHex, totalCount, parentLabelId, viewAll, teamLabelList) бодитоор агуулдаг — 2026-07-07-нд live баталгаажсан.

## API contract (`client.ts:107-124`)

| Үйлдэл | Method + Path | Payload |
|---|---|---|
| `getLabelList` | GET `/public-api/v2/contract-label` | — |
| `createLabel` | POST `/public-api/v1/contract-label` | `{name, parentLabelId?}` |
| `renameLabel` | PUT `/public-api/v1/contract-label` | `{id, name, parentLabelId}` |
| `updateLabelColor` | PUT `/public-api/v1/contract-label/update-color` | `{id, name, colorHex}` |
| `deleteLabel` | DELETE `/public-api/v1/contract-label/{id}` | — |
| `updateLabelMembers` | POST `/public-api/v1/team-label/update-by-team-id` | доор §Хуваалцах |
| `setLabel` / `unsetLabel` | POST `/public-api/v2/contract-action/set-label\|unset-label` | `{labelId, actionIdList}` |

**BFF дүрэм (чухал):** v1 catch-all (`src/app/backend/public-api/v1/[...path]/route.ts:16-26`) нь PUT/PATCH/DELETE-г **жинхэнээр дамжуулдаг**; харин v2 route (`.../v2/[...path]/route.ts:14-16`) дээр PUT/PATCH/DELETE = `okNoop`. Тиймээс label-ын бүх бичих үйлдэл v1 base-ээр явах ёстой — уншилт (`getLabelList`, set/unset-label) л v2. PUT round-trip upstream хүртэл live баталгаажсан.

## Query давхарга (`queries.ts`)

- `useLabelList` — queryKey `["contract-labels"]` (`queries.ts:6-8`).
- Нэг `useLabelMutation` helper (`queries.ts:235-242`) — **мутаци бүр `["contract-labels"]`-ийг invalidate хийнэ** (v2-ийн хуучирсан-cache алдааг давтахгүй): `useCreateLabel`, `useRenameLabel`, `useUpdateLabelColor`, `useDeleteLabel`, `useUpdateLabelMembers` (`queries.ts:245-257`).
- `useSetLabel`/`useUnsetLabel` (`queries.ts:260-283`) — received-list **болон** contract-labels хоёуланг invalidate (жагсаалтын мөр + sidebar-ын `totalCount` зэрэг шинэчлэгдэнэ).

## Received-list-ийн labelId шүүлт (business rule)

- Backend нь `GET /public-api/v2/contract-request/received-list` дээрх `labelId` параметрийг **шүүж өгдөг** — live баталгаажсан (нийт 243 → 6). v2-ийн тусдаа `/v3/contract-request/label-list` endpoint болон `BACKEND_URL_V3` base **шаардлагагүй** — v3-д байхгүй хэвээр.
- Урсгал: `labelId` searchParam (`ReceivedContent.tsx:31`) → queryKey-ийн хэсэг (`:155`) → `labelId !== "all"` үед request param (`:174-176,188-190`).
- Идэвхтэй мөр = `/documents/received` дээрх `labelId` searchParam (`nav-labels.tsx:332`).
- Toolbar chip-ууд (`received-content/ReceivedToolbar.tsx:36-47`): "Бүх хавтас" + `useLabelList`-ийн root хавтаснууд (cache-ээс, нэмэлт request үгүй). **Hydration дүрэм:** query cache SSR-ээс хойш дүүрсэн байж болох тул chip-үүд `useSyncExternalStore` mounted gate-ээр зөвхөн client дээр нэмэгдэнэ.

## Хуваалцах (org sharing)

- **Байрлалын дүрэм:** `ShareLabelDialog` нь `src/components/layout/`-д (features/documents-д БИШ) — eslint boundaries нь feature→feature import-ыг **public index-ийг оролцуулаад** хориглодог; layout (app-component) давхарга нь `features/profile` (`useTeamList`, `useEmployeePage`) болон `features/documents`-ыг хоёуланг нь import хийж болно.
- **Gating (v2 дүрэм):** зөвхөн root хавтас (`parentLabelId` байхгүй, `nav-labels.tsx:178`) + `selectedProfile.type === ORGANIZATION` + `role !== MEMBER` (`nav-labels.tsx:326-329`). Dialog нь **зөвхөн нээлттэй үедээ mount** болдог (`nav-labels.tsx:382-388`) — иргэн (CITIZEN) хэрэглэгчид team/employee query огт ажиллахгүй.
- **Payload contract** (`client.ts:148-155`, POST `/v1/team-label/update-by-team-id`): нэмэгдсэн → `teamIdList`/`employeeIdList`; хасагдсан → `teamLabelIdList` (серверээс ирсэн `TeamLabelEntry.id`); `viewAll` switch. Нэг endpoint л хэрэглэнэ — v2-ийн `create-by-team-id`/`remove-team-from-label` порт хийгдээгүй (v2-ийн модал ч зөвхөн update ашигладаг байсан). Round-trip нь ORGANIZATION profile дээр live баталгаажсан (нэмэх + хасах хоёулаа).
- Одоо хуваалцсан гишүүд `teamLabelList`-ээс checked seed хийгдэнэ; өөрчлөлтгүй үед Save идэвхгүй (`ShareLabelDialog.tsx:91-96,212`).

## `/company-employee` vs `/company-employee/employee-list` (анхаар — түгээмэл алдаа)

- `GET /v1/company-employee` (`features/profile/api/client.ts:516`, `useCompanyEmployees`) нь **тухайн ХЭРЭГЛЭГЧИЙН** компани бүр дэх өөрийн бүртгэлүүдийг буцаадаг (`user` object байхгүй, ижил email × N) — байгууллагын ажилтнуудын жагсаалт **БИШ**.
- Байгууллагын ажилтнууд = `GET /v1/company-employee/employee-list` (хуудаслалттай, `user` мэдээлэлтэй; `client.ts:595`, `useEmployeePage` `queries.ts:526`). v2-ийн workerList ч үүнийг хэрэглэдэг.
- `ShareLabelDialog` нь `useEmployeePage(1, 100)` хэрэглэнэ (`ShareLabelDialog.tsx:65-67`); идэвхтэй (`active`) ажилтнуудыг л харуулна.

## Drag & drop (native HTML5, dependency-гүй)

v2-ийн react-dnd-г **native HTML5 DnD**-ээр орлуулсан:
- `lib/drag.ts` — `CONTRACT_DRAG_TYPE = "application/x-egeree-contract-actions"` MIME; `contractActionIds(contract)` = тухайн хэрэглэгчийн төлөөлдөг action id-ууд (`relatedParticipantKeyList` байвал шүүнэ — LabelModal-тай ижил дүрэм, LabelModal ч үүнийг хэрэглэдэг).
- Shared `DataTable` нь optional `rowProps` prop-той (`src/shared/ui/components/data-table/index.tsx:59,166`) — мөр бүрт DOM props тарааж өгнө. `ReceivedContent.tsx:215-224` үүгээр `draggable` + `onDragStart` (actionIdList JSON, `effectAllowed = "copy"`) тохируулна.
- Drop target = `nav-labels.tsx`-ийн мөр бүр, гүн харгалзахгүй: `onDragOver` нь зөвхөн `CONTRACT_DRAG_TYPE`-д preventDefault + ring highlight (`:261-268`), drop → `useSetLabel` (`handleDrop`, `:220-237`).

## UI бүтцийн тогтмол шийдвэрүүд (`nav-labels.tsx`)

- **Бүх гүнд `SidebarMenuItem`** (`SidebarMenuSubItem/SubButton` биш) — `SidebarMenuAction`-ы hover төлөв зөвхөн `group/menu-item` дээр ажилладаг тул sub-мөрөнд kebab гарахгүй байсан.
- Мөр бүр өөрийн `group/label-row` wrapper-тай (`:241`) — kit-ийн `group/menu-item` hover нь nested модонд эцгээс хүүхэд рүү cascade хийдэг.
- **Тоо ↔ kebab нэг байрлалд ээлжилнэ** (`:281`): энгийн үед `totalCount`, hover/цэс нээлттэй/kebab focus-visible үед kebab. has-selector нь `[aria-haspopup=menu]`-д чиглэсэн — chevron CollapsibleTrigger-ийн `data-state=open`-д тоо нуугдах ёсгүй.
- Chevron зүүн талд (`left-1`), бүх мөр `pl-7` gutter-тэй; sub-мод `SidebarMenuSub mr-0 pr-0` (баруун талын шаталт арилгасан).
- Hover bg = хавтасны өөрийн өнгө 15%-иар (`--label-color` CSS var, `:255-259`); **`!` suffix шаардлагатай** — twMerge нь `bg-[color:var(...)]`-ыг kit-ийн `hover:bg-sidebar-accent`-тэй dedupe хийдэггүй, kit rule нь stylesheet-д хожуу байрладаг.
- Нэр тайрагдсан үед л tooltip: mouseenter дээр `scrollWidth > clientWidth` шалгаад dynamic `title` (`:270-274`).
- Loading skeleton нь **тогтмол өргөнтэй** (`:346-354`) — `SidebarMenuSkeleton`-ы `Math.random()` өргөн SSR hydration mismatch өгдөг.
- Өнгөгүй хавтасны нэгдсэн fallback = `var(--muted-foreground)` (`:60`; v2-д хоёр өөр fallback холилдож байсан).
- Өнгө сонгогч = 10 preset swatch, `DropdownMenuItem` хэлбэрээр (`:63-74,157-171`) — сонгоход цэс өөрөө хаагдана; react-color dependency-гүй.
- Root хавтас үүсгэх `+` = `SidebarGroupAction` (`:337-343`) — хавтас байхгүй үед ч group нуугдахгүй (`+` хүрэх боломжтой байх ёстой).
- Create/Rename dialog-ууд `aria-describedby={undefined}` (`CreateLabelDialog.tsx:61`, `RenameLabelDialog.tsx:63`) — Radix-ийн "Missing Description" warning-оос сэргийлнэ. Rename нь render-time compare-and-set-ээр одоогийн нэрийг seed хийнэ (`RenameLabelDialog.tsx:33-36`).

## i18n

**Шинэ түлхүүр нэг ч нэмээгүй** — 4 locale (`src/i18n/languages-data/{mn,en,cn,kr}.json`) бүгд бэлэн байсан:
- `sidebar.folder` — group-ын гарчиг "Хавтас" (`mn.json:109`).
- `template.*` — CRUD dialog/цэсний бүх текст: `createFolder`, `createSubfolder`, `changeFolderName`, `changeFolderColor`, `deleteFolder`, `folderName`, `inputFolderName` г.м. (`mn.json:442`-оос).
- `label.*` — ShareLabelDialog: `addMember`, `seeAllMembers`, `successChangeLabelMembers` г.м. (`mn.json:1609`-өөс).
