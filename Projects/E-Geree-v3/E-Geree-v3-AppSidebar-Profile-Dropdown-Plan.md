---
title: AppSidebar Profile Dropdown — Design
status: done
date: 2026-07-01
tags: [plan, design, e-geree-v3, app-sidebar, profile]
---

# AppSidebar Profile Dropdown — Design

> [!warning] Аудит 2026-07-07 (Claude) — дутуу зүйлс
> **Ерөнхий төлөв:** mostly-done (9/10 хэрэгжсэн)
> **Дутуу / хийгдээгүй:**
> - ⚪ Manual UI check (browser walkthrough) — plan line 69 "Not yet manually clicked through in browser — do that before merging" хэвээр атал бүх commit dev-khishigee дээр merged; 4 зүйлт checklist (recent list шинэчлэл, section хил, dialog search, F1 shortcut)-ийг бүрэн гүйцэтгэсэн бичлэг worklog-уудад алга. Хэсэгчилсэн эсрэг баримт: a4095d8-д dialog-ийг browser дээр дарж bug засcан, 07-07 worklog-д live profile switch хийсэн. Functional эрсдэл бага (unit test 4/4 PASS).
> **Тэмдэглэл:** Бүх код талын ажил (useRecentProfiles, buildProfileSections, AllProfilesDialog, dropdown дахин бичилт, userId threading) хэрэгжиж dev-khishigee-д merged, unit test 4/4 PASS. Frontmatter-ийн "approved, not implemented" status хуучирсан тул "implemented/merged" болгож засах, мөн manual UI checklist-ийг нэг удаа бүрэн гүйлгэх эсэхийг шийдэх нь зүйтэй.

## Context
- File: `src/components/layout/app-sidebar.tsx`
- Profile switcher: `src/shared/ui/components/profile-dropdown-content.tsx`
- Data model: `src/types/auth-types.ts` (`Profile`, `AuthDataSuccess`)
- Redux: `src/store/auth-slice.ts`
- Current state: flat `DropdownMenuItem` list of all profiles, no recent/pinned structure.

## Requirements
- Personal (citizen, `LegalEntityType.CITIZEN`) profile pinned at top, always. Never appears in recent/other lists.
- Recent profiles section: last 3 used (org profiles only, MRU order).
- Other profiles section: next up to 5 (non-recent org profiles).
- "Бүх профайл харах" button opens full profile list (search).
- Anything beyond pinned + recent(3) + other(5) — only reachable via the full list.

## Decisions (from brainstorming Q&A)
- Personal profile = citizen profile, always pinned separately, excluded from recent/other.
- Recent profiles storage: **localStorage** (not Redux, not backend) — device-local, same pattern as `src/features/contract-create/store/persist.ts`.
- localStorage key scoped per user: `recent_profiles_${userId}` — prevents recent-list bleed across accounts on a shared browser/device.
- Full profile list UI: **Dialog + Command** (shadcn, both already in `components/ui`) with search — not a new route, not a Sheet.

## Architecture
- New hook `useRecentProfiles(userId)`: reads/writes localStorage key `recent_profiles_${userId}`. Array of `profile.value`, max 3, MRU (unshift + dedupe + slice). Guards SSR (`typeof window`) and JSON parse errors (try/catch → empty array fallback).
- Hooked into successful profile switch (existing `switchProfile` callback in `profile-dropdown-content.tsx`, and the new dialog's switch handler) — on success, push `profile.value` into the recent list via the hook's update fn.
- Pure helper `buildProfileSections(profileList, selectedProfile, recentIds)` → `{ personal, recent, other, rest }`:
  - `personal` = profile with `type === LegalEntityType.CITIZEN`
  - `recent` = up to 3, from `recentIds ∩ profileList` (drop stale/revoked ids silently)
  - `other` = next up to 5 org profiles not already in `recent`
  - `rest` = everything else, only shown in the full-list dialog

## Components
- Rewrite `ProfileDropdownContent` (`src/shared/ui/components/profile-dropdown-content.tsx`): sections in order — Хувийн профайл (pinned, 1) → Сүүлд ашигласан (≤3) → Бусад профайл (≤5) → "Бүх профайл харах" trigger item.
- New `AllProfilesDialog` component: Dialog + Command primitives (already in `src/components/ui/`). Full-text search over `profileList`, click/select switches profile via existing `switchProfileAction`, closes dialog.
- Standard shadcn pattern to avoid the dropdown-closes-dialog race: trigger `DropdownMenuItem` uses `onSelect={(e) => e.preventDefault()}` then sets dialog open state.

## Data flow
`profileList` / `selectedProfile`: existing Redux `auth-slice` (no change). Recent ids: localStorage via the new hook, recomputed after each successful switch. No backend/API change needed.

## Error handling
- localStorage unavailable or corrupt JSON → empty recent list, UI degrades gracefully to pinned personal + other list (old flat behavior minus recent section).
- Recent id referencing a profile no longer in `profileList` (access revoked) → filtered out silently, not shown as a broken entry.

## Testing
- Unit test `buildProfileSections` (pure function, no React needed) — cases: <4 total profiles, >8 profiles (overflow to rest/dialog), fresh user with empty recent list, stale recent id filtered out.
- Manual UI check: profile switch updates recent list, section boundaries correct, dialog search works, F1 citizen shortcut still works.

## Status
Design approved by user in brainstorming session (2026-07-01). Implemented same day — see Processing log below.

## Processing log (2026-07-01)

Branch: `feat/profile-dropdown-recent` (off `dev-khishigee`).

1. `feat(profile): add buildProfileSections helper` (`8ae4e67`) — pure personal/recent/other/rest split, co-located vitest test (`src/shared/lib/profile-sections.ts` + `.test.ts`). 4/4 passing.
2. `feat(profile): add useRecentProfiles hook` (`44731a1`) — `src/hooks/useRecentProfiles.ts`, localStorage key `recent_profiles_<userId>`, MRU capped at 3, SSR/parse-safe.
3. `feat(profile): add AllProfilesDialog` (`ab4d7bb`) — `src/shared/ui/components/all-profiles-dialog.tsx`, built on the existing `CommandDialog` primitive (search + select), reuses the switch hook below instead of duplicating switch logic.
4. `feat(profile): restructure dropdown into personal/recent/other sections` (`1d4ce01`) — extracted switch logic (previously inline in `ProfileDropdownContent`) into shared `src/hooks/useProfileSwitch.ts` so the dropdown and the new dialog don't duplicate the `switchProfileAction` + Redux + router.refresh flow. Rewrote `ProfileDropdownContent` into personal (pinned) → Сүүлд ашигласан (≤3) → Бусад профайл (≤5) → "Бүх профайл харах" trigger opening the dialog.

Deviation from the original design doc: also fixed a `react-hooks/set-state-in-effect` lint in `useRecentProfiles` by adjusting state during render (compare `loadedFor` vs `userId`) instead of a `useEffect` — same outcome (recent list reloads when the signed-in user changes), just the React-endorsed pattern for derived state reset instead of an effect.

Verification: `tsc --noEmit` clean, `eslint` clean on all touched/new files, full `vitest run` 28/28 passing. Not yet manually clicked through in browser — do that before merging (profile switch updates recent list, section boundaries, dialog search, F1 citizen shortcut).

6. `refactor(profile): thread userId through dropdown/dialog, restyle items` (`ffdf461`) — `ProfileDropdownContent`/`AllProfilesDialog` now take an explicit `userId` prop instead of deriving `personalId` by scanning `profileList` for the CITIZEN entry; `useProfileSwitch` keys off `userId` directly. **Deviation from the note below**: this DID require touching the two consumers — `profile-switch.tsx` passes `userId={user.id}`, `team-switcher.tsx` passes `userId={data.user.id}`. Also restyled list items (avatar `size-6`→`size-7` + active ring, active-row `Check` icon, `DropdownMenuSeparator` before "Бүх профайл харах").
