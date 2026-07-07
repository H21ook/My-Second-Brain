# 2026-07-07

## Contract Detail — Phase 2d-3 (GSIGN digital signature)
- Continued [[E-Geree-v3-Contract-Detail-Page-Phase2d-Plan]]. 2d-2a/2b/2c already done (`8dd7531`). Resumed the deferred 2d-3.
- **User decisions:** scope = **GSIGN only** (DAN/e-Mongolia phone push); TRIDUM local-client → deferred as **2d-3b**. Test = **code-confidence + UI-to-push** (no real signature completed).
- **Understand-phase workflow** (6 parallel readers, ultracode) mapped v2 source + v3 targets. Key findings:
  - Plan stale: `FlowActionDock.tsx` (commit `9e6409b`) was deleted by `c2c07e5`, merged into `ActionButtons.tsx`. Do NOT recreate it. `onDigitalSign` hook already wired (default noop) — ActionButtons needed NO change.
  - Digital-signature upstream = **same host as v2** (`digital-signature-api.e-geree.mn/…/v1`) → v2 is source-of-truth, low protocol risk.
  - Envelope is faithful: `coreFetcher` builds `{isOk: res.ok, data: rawBody}` by HTTP status — identical to v2's `httpRequest`. So v2's `data.X` field access → v3's `.X` after unwrap.
  - **v3 had NO socket infra** — built from scratch. Installed `socket.io-client@^4` (v2 = `^4.7.5`, server is socket.io — matched major).

- **Built (7 changes, GSIGN):**
  - `core/config`: `getDigitalSignatureUrl()`.
  - **New** `app/backend/digital-signature/[...path]/route.ts` (BFF proxy, POST+GET; base version-baked so client path adds no extra version).
  - `documents/api/client.ts`: `createSignRequest` + `sendGSignRequest` + payload/result types.
  - `documents/api/queries.ts`: `useStartGSign` — combines create+g-sign into one mutation, returns socket room key; guards missing `otp/inputPdfUrl/id`. No invalidate here (socket finalizes).
  - **New** `documents/components/detail/DigitalSignModal.tsx` — phone (preset from `userProfile.mobile`) → push → waiting-for-socket. Fixed two v2 bugs: module-level socket leak → `useEffect` cleanup `disconnect()`; hang-on-transport-failure → `connect_error` handler.
  - `ContractDetail.tsx`: `handleDigitalSign`/`handleDigitalSigned` (invalidate detail + toast on socket success), wired `onDigitalSign`, rendered modal.

- **eslint discoveries:** writing `ref.current` during render → `react-hooks/refs` (moved to effect). Synchronous `setState` in `useEffect` → `react-hooks/set-state-in-effect` (used the codebase's render-time compare-and-set `prevOpen` idiom instead, same as ContractDetail `seededFieldList`).

- **Verify:** tsc 0 · eslint 0 (new/touched; only pre-existing `CopyIconButton` warning in ContractDetail, from 2d-2c, not mine) · knip no new complaints · vitest 28/28 · **`npm run build` exit 0** (route registered, socket.io-client bundles clean). **NOT** browser-drive-verified (needs a `DIGITAL_SIGNATURE_PENDING` contract; completing it = real signature).

- **Committed** `1ac18a2` (8 files, +403/-1) on `dev-khishigee`.
- **DEPLOY note:** `DIGITAL_SIGNATURE_URL` + `NEXT_PUBLIC_DIGITAL_SIGNATURE_SOCKET_URL` env needed in prod (.env gitignored).

## Documentation impact
- `E-Geree-v3-Networking-BFF.md` — a new upstream (`getDigitalSignatureUrl` + `/backend/digital-signature` proxy) was added; worth a one-line note there like the payment proxy entry. Not yet updated.
- Otherwise no architecture/routing/state changes.

## Label sidebar — Phase A (NavProjects → NavLabels)

- Executed Phase A of Label-Sidebar-Migration-Plan (тогтмол мэдлэг: [[E-Geree-v3-Label-Sidebar]]) (Phase B CRUD not started; needs B1 backend PUT/DELETE check first).
- **Built:**
  - `LabelNode` extended with `colorHex?/totalCount?/parentLabelId?` on `contractLabel` (`documents/api/client.ts`).
  - **New** `components/layout/nav-labels.tsx` — `useLabelList` tree: Collapsible (default-open, same as LabelModal), color dot (single fallback `var(--muted-foreground)`), inline count, active state = `labelId` searchParam on `/documents/received`, `SidebarMenuSkeleton` while loading, group hidden when no labels. Links via `@/i18n/navigation`. Group label = existing `sidebar.folder` key (all 4 locales already had it — **no new i18n keys needed**).
  - `app-sidebar.tsx`: `<NavProjects>` → `<NavLabels>`; deleted `nav-projects.tsx` + demo `projects` data + unused icons.
  - **A6 wiring:** `ReceivedContent` now passes `labelId` into the received-list queryKey + request params (`labelId` param when ≠ "all"). `ReceivedToolbar` label chips: deleted hardcoded `labels.ts` stub, chips now = "Бүх хавтас" + real root labels from `useLabelList` (cached, no extra request), clickable → sets/clears `labelId` searchParam.
- **Key discovery:** v2 label filtering used a separate endpoint `GET {BACKEND_URL_V3}/contract-request/label-list?labelId=` — v3 has **no v3 upstream base or BFF route**. Wired `labelId` straight into v2 received-list instead; if backend ignores it, fallback per plan §7 = add BACKEND_URL_V3 + `/backend/public-api/v3` proxy + label-list endpoint.
- **eslint boundaries note:** `app-component` (layout) → `feature` import is allowed (default allow; only feature→feature and shared→feature are restricted) — plan's fallback (public index.ts re-export) not needed.
- **Verify:** tsc 0 · eslint 0 errors (2 pre-existing warnings, unrelated files) · vitest 30/30. **NOT browser-verified**: whether `colorHex/totalCount` actually arrive in `getLabelList` response and whether backend honors `labelId` on received-list needs a live session check (plan risk §7).
- Committed `9734fe9` (7 files, +192/−139).

## Label sidebar — Phase B (CRUD)

- **B1 static check:** `/backend/public-api/v1/[...path]` proxies PUT/PATCH/DELETE for real (okNoop is only on the v2 route) — all label CRUD is v1 base, so the plan's okNoop risk is cleared on the BFF side. Upstream behavior still not live-verified.
- **Built:**
  - `client.ts`: `createLabel {name, parentLabelId?}` POST · `renameLabel {id, name, parentLabelId}` PUT · `updateLabelColor {id, name, colorHex}` PUT `/update-color` · `deleteLabel` DELETE `/{id}` (payload shapes read from v2 modals, source of truth).
  - `queries.ts`: one `useLabelMutation` helper → `useCreateLabel/useRenameLabel/useUpdateLabelColor/useDeleteLabel`, all invalidate `["contract-labels"]`.
  - `nav-labels.tsx` rework: **rows unified to `SidebarMenuItem` at every depth** (dropped `SidebarMenuSubItem/SubButton`) because `SidebarMenuAction showOnHover` only reacts to `group/menu-item` hover — sub-rows would never show the kebab. Kebab right-1, chevron moved to `right-6` on children rows, count margin shifts (`mr-5`/`mr-11`). Root create = `SidebarGroupAction` plus button (group no longer hidden when label list is empty — the + must stay reachable).
  - Dialogs (features/documents/components, PascalCase): `CreateLabelDialog` (root+sub via `parent` prop), `RenameLabelDialog` (button says "Хадгалах" — v2's "үүсгэх" cosmetic bug fixed; render-time compare-and-set seeding), `DeleteLabelDialog` (alert-dialog, mirrors DeleteConfirmDialog). Copy ID = `navigator.clipboard` + sonner toast, no modal.
  - Color picker: 10 preset swatches in `DropdownMenuSubContent` grid, active color gets a ring. No react-color dependency.
- **i18n:** zero new keys — everything needed already in `template.*` across all 4 locales (createFolder, createSubfolder, changeFolderName, changeFolderColor, deleteFolder, folderCreate, renamedFolder, mainFolderName, subfolderName, inputFolderName, folderName).
- **Deferred (Phase C per plan):** member/org sharing (`team-label/update-by-team-id`, was ORGANIZATION-only in v2's dropdown), drag&drop.
- **Verify:** tsc 0 · eslint 0 errors (same 2 pre-existing warnings) · vitest 282/282 (test count jumped from 30 — new test files landed from the parallel payment session, all green) · `npm run build` exit 0. **NOT browser-verified** — CRUD round-trips need a live session.
- Committed `ca48ace` (6 files, +481/−50).

## Label sidebar — Phase C (sharing + drag&drop)

- **Member/team sharing:**
  - `ShareLabelDialog` placed in `src/components/layout/` (NOT features/documents) — eslint boundaries forbids feature→feature imports **including the public index** (the rule's own error message suggests index.ts but the disallow pattern matches every file under `src/features/*`); layout (app-component) may import both `features/profile` (useTeamList, useCompanyEmployees — Phase G2 infra was already there) and `features/documents`.
  - Payload parity with v2 AddMemberToLabelModal: `{labelId, viewAll, teamIdList (added teams), employeeIdList (added employees), teamLabelIdList (removed → teamLabelId)}` to POST `/v1/team-label/update-by-team-id`. v2's other two team-label endpoints (create-by-team-id, remove-team-from-label) not ported — v2's own modal only used update.
  - `LabelNode.contractLabel` += `viewAll?`, `teamLabelList?` (`TeamLabelEntry` = team XOR employee, `id` = teamLabelId used for removal).
  - Dropdown item gated: root label + `selectedProfile.type === ORGANIZATION` + `role !== MEMBER` (v2 rule). Dialog mounted only while open so profile queries never fire for citizens.
  - UI simplified vs v2's 528-line modal (no avatars/type-filter dropdown; single searchable checkbox list, teams then employees) — payload and gating are the parity that matters.
- **Drag&drop (no new dependency):** native HTML5 DnD replaces v2's react-dnd.
  - `features/documents/lib/drag.ts`: `CONTRACT_DRAG_TYPE` MIME + `contractActionIds()` (extracted from LabelModal, which now consumes it too; 3 vitest cases).
  - Shared `DataTable` got an optional `rowProps` prop (spread onto TableRow, className merged) — ReceivedContent uses it for `draggable` + `onDragStart` (sets actionIdList JSON).
  - `nav-labels` rows are drop targets at every depth: dragOver ring highlight, drop → `useSetLabel`. `useSetLabel`/`useUnsetLabel` now also invalidate `contract-labels` so sidebar `totalCount` refreshes.
- **Verify:** tsc 0 · eslint 0 errors · vitest 287+3 all green · build exit 0. **NOT browser-verified** — sharing round-trip and DnD need a live ORGANIZATION session.
- Committed `862af55` (9 files, +382/−17).

## Label sidebar — LIVE verification (chrome-devtools on localhost:3000 dev server, logged-in CITIZEN session)

All plan risks resolved except org sharing:
- **getLabelList response** carries everything the types assumed: `colorHex, totalCount, parentLabelId, viewAll, teamLabelList` (plus createdDate etc.) — verified via in-page fetch.
- **received-list `labelId` filter IS honored by backend**: total 243 → 6 with `labelId=` param. No need for the v2 `/v3/contract-request/label-list` endpoint or a BACKEND_URL_V3 proxy — plan §7 fallback can be dropped.
- **Full CRUD round-trip performed in UI**: create "Claude test хавтас" (sidebar + toolbar chip appeared instantly — invalidate works) → rename to "Claude renamed" (dialog seeds current name; **PUT via v1 BFF proxy works upstream** — okNoop risk fully dead) → color swatch #3b82f6 (dot updated) → delete via alert-dialog (sidebar + chip gone, no error toast). Copy-ID clicked, no console error (clipboard read-back not testable — permission prompt hangs CDP evaluate).
- **DnD works**: synthetic DragEvent on a table row produced dataTransfer type `application/x-egeree-contract-actions` with the 2 action ids; drop on a label fired setLabel → label count 0→1 and row left the filtered view (both invalidations firing). Test contract moved back to its original label afterwards (set-label via fetch, verified back: 6 rows).
- **Sharing gating verified negatively**: CITIZEN profile → "Гишүүн нэмэх" absent from dropdown, no team/employee requests fired. **Positive path (dialog + update-by-team-id round-trip) still untested — needs an ORGANIZATION profile.**
- **Fix from console audit**: Radix warning "Missing Description for DialogContent" ×4 → added `aria-describedby={undefined}` to Create/Rename label dialogs.
- Minor known UX quirk: color swatch click leaves the dropdown open (v2's picker behaved similarly); left as-is.

## Label sidebar — live UX review round (user-driven, committed `8b9f892`)

User reviewed in browser alongside the session (switched to HADES LLC ORGANIZATION profile — share menu item gating confirmed positive side too). Fixes, each live-verified:
- Chevron front gutter (pl-7 all rows), tree default-closed; sub-tree right-edge stepping fixed (`SidebarMenuSub mr-0 pr-0`).
- Count ↔ kebab swap in ONE slot (verified identical rects): rest = count, hover/menu-open/focus-visible = kebab. Key gotchas discovered: (1) kit `group/menu-item` hover cascades into nested rows → per-row `group/label-row` wrapper; (2) `group-has-data-[state=open]` also matches the chevron CollapsibleTrigger → has-selector must target `[aria-haspopup=menu]`; (3) Radix returns focus to trigger after outside-click close → count must also hide on trigger `:focus-visible` or both show.
- Hover bg = label's own color at 15% (`--label-color` var + color-mix). twMerge does NOT dedupe `bg-[color:var(...)]` vs kit's `hover:bg-sidebar-accent`, and the kit rule sorts later in the utilities layer → needed `!` suffix.
- Color swatches converted to DropdownMenuItem (auto-close on select) + success toast. Menu got "Хавтасны тохиргоо" title (user edited wording). Labels shortened: "Өнгө солих", "Гишүүдийн тохиргоо".
- Truncation-only tooltip via dynamic `title` on mouseenter (scrollWidth check).
- Hydration fix #2: toolbar chips differ between SSR HTML and late Suspense hydration (query cache already filled) → `useSyncExternalStore` mounted gate. (#1 was the Math.random skeleton, same day.)
- During CDP testing accidentally changed two real label colors; both restored via update-color API (#AB4ABA).

## ShareLabelDialog fix + sharing round-trip verified (`bb966e9`, pushed)

- **Wrong endpoint found**: `/company-employee` returns the CURRENT USER's employee records across all their companies (no `user` object, same email × N) — NOT the company's staff. v2's workerList actually uses `/company-employee/employee-list` (paginated). Switched to existing `useEmployeePage(1, 100)`. **Same bug exists in profile's EmployeeSelector (uses useCompanyEmployees) — not fixed, separate scope.**
- Teams/members split into sections with sticky headers (needed `z-10` — content painted over the sticky label); team rows show member count.
- **Sharing round-trip live-verified on HADES LLC (ORGANIZATION)**: existing shared member seeded checked from `teamLabelList`; add → `employeeIdList`, remove → `teamLabelIdList`, both via `update-by-team-id` succeeded (1→2→1 entries). Save disabled when unchanged. The plan's last unverified path is now closed.
- All label-sidebar work pushed to `origin/dev-khishigee` (A `9734fe9` · B `ca48ace` · C `862af55` · polish `8b9f892` · share fix `bb966e9`).

## Documentation impact (label sidebar phases A–C)
- Plan doc `Label-Sidebar-Migration-Plan.md` updated to `status: done` with per-phase notes — can be deleted per its own header note once live-verified.
- No architecture/routing/state doc changes (no new upstream, no Redux, TanStack Query only).

## Нээлттэй TODO (Untitled.md-ээс нэгтгэв, 2026-07-07)

- [ ] **Create template шалгах** — template үүсгэх урсгалыг live session-д шалгаж эцэслэх (нээлттэй plan-debt: template flow live check).
- [ ] **Received list дутуу гүйцээх** — received list-ийн дутуу зүйлсийг гүйцээх (labelId filter 2026-07-07-нд live-баталгаажсан ч бусад gap үлдсэн).
- [ ] **Template list** — жагсаалтын хуудсыг шалгаж дутууг гүйцээх.
- [ ] **Sent list** — жагсаалтын хуудсыг шалгаж дутууг гүйцээх.
- [ ] **Contract detail** — live баталгаажуулалт: 2d-3 GSIGN browser-drive-verified хийгдээгүй (`DIGITAL_SIGNATURE_PENDING` гэрээ шаардлагатай).
