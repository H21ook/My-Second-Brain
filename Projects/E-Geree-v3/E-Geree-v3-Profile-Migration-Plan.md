---
title: Profile Section Migration — v2 → v3
status: 'A+B1+B2+C+E+F+G1+G2 committed (G2 = 566b7ca, 2026-07-04); D deferred — G split: G1 ✅ G2 ✅ → G3 dashboard+subscription (recharts 2.15.4 confirmed); api-connection stays deferred. Post-G2 polish commits (2026-07-06) logged in §11 "Post-G2 unplanned work". 2026-07-07 gap-audit: deferred ledger §16, smoke gate §17, deploy checklist §18, G3 pre-flight in §11. Next: G3 in a FRESH chat (haiku recon → Sonnet subagent impl → Fable main-thread verify) — resolve payment-reuse + OrgVerificationHeader decisions FIRST (§16)'
date: 2026-07-03
updated: 2026-07-07
tags: [plan, migration, e-geree-v3, profile]
---

# Profile Section Migration (v2 → v3)

> [!warning] Аудит 2026-07-07 (Claude) — дутуу зүйлс
> **Ерөнхий төлөв:** mostly-done (12/23 хэрэгжсэн — A–G2 код бүрэн, G3 + gate-ууд үлдсэн)
> **Дутуу / хийгдээгүй:**
> - 🔴 Phase G3 — `organization/dashboard` + `organization/subscription` огт эхлээгүй: route dir байхгүй, nav.ts:85,103 `enabled: false`, G3 commit алга
> - 🔴 G3 pre-flight #1 — payment-modal reuse шийдвэр (hoist to `shared/` vs duplicate) гараагүй, G3-г блоклоно (§16 мөр 364 ❌; ff09d0b commit modal-ыг documents дотор self-contained үлдээсэн)
> - 🔴 §17 Manual smoke gate — 8 checkbox-ын НЭГ ч хийгдээгүй (B2/C auth-critical, E payment, G1 org delete live тест); зөвхөн B1 SMS урсгал bf4e7b6-ээр хэсэгчлэн live-validated
> - 🟡 Subscription attach point-ууд — profile-header banner, SubscriptionPlan card, user-manage-ийн disabled товч (SubscriptionPlanCard.tsx:60 `disabled` + "Тун удахгүй" хэвээр, §16 ❌→G3)
> - 🟡 OrgVerificationHeader — migrate-in-G3 vs defer шийдвэр гараагүй (§16 мөр 360, v3-д 0 файл)
> - 🟡 ProfileNav mobile fallback — `w-56` fixed, responsive class 0; drawer/dropdown UX шийдвэр гараагүй (§16 мөр 365)
> - 🟡 §18 deploy/infra — PAYMENT_URL (+шинээр PAYMENT_URL_V2) env out-of-band баталгаагүй, шинэ BFF surface security review хийгдээгүй
> - ⚪ Phase D — user/signature бүхэлдээ (draw/crop/HEIC/bg-remove); §16-д tracked deferral, package-ууд суугаагүй
> - ⚪ Бусад tracked deferral: user/payment stub, user/settings, api-connection, DAN linking, StampCreateModal, VAT/receipt modal — бүгд §16-д эзэн/trigger-тэй
> **Тэмдэглэл:** A–G2-ийн 15 commit бүгд dev-khishigee дээр кодоор баталгаажсан; frontmatter status кодтой 100% таарч байна. Шинэ ажиглалт: Worklog 07-07-д илэрсэн EmployeeSelector-ийн `/company-employee` буруу endpoint bug §16 ledger-т 2026-07-07-нд нэмэгдсэн (owner: G-phase follow-up). Дараагийн алхам: payment-reuse + OrgVerificationHeader шийдвэрийг ЭХЭЛЖ гаргаад G3.

## 1. Title
Migrate `/profile` section from e-geree-v2 to e-geree-v3.

## 2. Date/time
Discovery + plan: 2026-07-03 evening. Implementation: 2026-07-03→04 (phases A–G2 committed), post-G2 polish 2026-07-06. Plan gap-audit + note reconciliation: 2026-07-07.

## 3. Source path
`e-geree-v2/src/app/[locale]/(protected)/(withSidebar)/profile` — plus its real body:
- `e-geree-v2/src/components/pages/profile/**` (68 files, 10,106 lines)
- `e-geree-v2/src/components/layout/protected/profile/{profile-sidebar (353L), profile-header (135L)}`
- `e-geree-v2/src/lib/actions/{payment,user,signature,legalEntity,userReferral,dashboard,environmental-statistic}.js`
- `e-geree-v2/src/lib/services/{auth,security,team-service,address}.js`

## 4. Target path
- Routes: `e-geree-v3/src/app/[locale]/(protected)/(with-sidebar)/profile/**` (route group name differs: `(with-sidebar)` not `(withSidebar)`; empty dir already exists)
- Feature: `e-geree-v3/src/features/profile/{api,components,hooks,lib,schema,types,index.ts}` (new)
- Translations: v3 locale message files, all 4 locales (mn, en, cn, kr)

## 5. v2 structure summary (discovery, 2026-07-03)
17 subpages under two branches, thin route files (886L total), heavy page components:

```
/profile → redirect → /profile/user/personal-information
/profile/user/{personal-information, security, signature, login-history,
               payment, payment-history, subscription, referral, settings, organization}
/profile/organization/{information, dashboard, teams, teams/[slug],
                       user-manage, subscription, api-connection}
```

- Nested profile sidebar (own nav, 353L) + profile header inside app layout → double-chrome.
- Data: Next server actions (`@/lib/actions/*`) calling upstream directly; server components fetch in body; client pages useEffect+useState.
- State: 7 React contexts (profile, auth, company, payment, toast, top-loader-router, org-verification).
- Auth gates: org pages blocked for CITIZEN profile; XYP company sync gated on `authUser.danVerified`; api-connection gated on subscription feature flag `FEATURE_USE_CONTRACT_CALLBACK`.
- All `.jsx`, untyped. No RHF/zod — manual controlled state.
- i18n namespaces used: `profileDropdown`, `actions`, `personalInfo`, `security`, `status` (v2 `src/i18n/lang/{en,mn,kr,cn}.json`).
- Biggest clusters (lines): security ~1,380; teams ~1,650; personal-information ~1,570; org-info ~1,300; signature ~900; user-manage ~970; user/organization route page 439 (monolith client component).

## 6. v3 structure summary
- Layering enforced by eslint-boundaries: `app → features → shared → core`; features expose only `index.ts`.
- Feature slice pattern (canonical: `features/documents`): `api/client.ts` (apiClient → `/backend` BFF), `api/queries.ts` (TanStack Query, key factories), `components/` (PascalCase), hooks camelCase, rest kebab-case.
- BFF mandatory: browser only calls `/backend/*`; generic public-api proxy exists (documents feature hits `${V1}/contract-request` through apiClient with no bespoke route handlers).
- Forms: RHF + Zod v4, error messages are i18n keys.
- Nav: `@/i18n/navigation` (next-intl), never raw next/navigation. Breadcrumbs via `useSetBreadcrumb`.
- State: Redux auth-slice already has `profileList`, `selectedProfile`, user; `useProfileSwitch` hook exists; sonner toast; TanStack Query for server cache.
- Profile switcher UI already built in v3 (dropdown + AllProfilesDialog) — v2's user/organization page overlaps with it.
- Route dir `(protected)/(with-sidebar)/profile/` exists, empty.

## 7. Migration plan (proposed phases)
Everything is rewrite, not copy: .jsx→.tsx, server-actions→apiClient+React Query, contexts→v3 equivalents.

- **Phase A — shell**: `profile/layout.tsx` + profile section nav (v3-styled, from v2 profile-sidebar 353L), `/profile` redirect page, `src/features/profile/` skeleton, breadcrumb, CITIZEN-vs-ORG gating via auth-slice.
- **Phase B — user/personal-information** (~1,570L): info view, ProfileImage, AddressInformation, modals: EmailChange, PhoneNumberChange, AddressChange, VerificationSteps, UserStampImage. RHF+zod for modal forms.
- **Phase C — user/security** (~1,380L): 2FA activate/deactivate, password change, anti-phishing, advanced config. Auth-critical → high-effort verify + manual smoke test.
- **Phase D — user/signature** (~900L): signature list, create modal, editor, drawer, slides, bg-remove.
- **Phase E — user misc** (~900L): login-history, payment-history, subscription, referral, settings — table pages → reuse v3 `shared/ui/components/data-table`.
- **Phase F — user/organization** (439L monolith): probably shrinks a lot — v3 already has profile switching; keep XYP sync + org grid/list; decompose into components + `useSyncOrganizations` hook.
- **Phase G — organization branch** (~4,600L): org-info (+stamp/logo/api-key/delete), dashboard (charts — chart lib TBD), teams (+CRUD modals, employee selector), user-manage (member list, employee add, contract transfer, subscription plan), subscription, api-connection.

Endpoint mapping done per phase: read v2 action file → exact upstream URL → map to `/backend/public-api/v1|v2` apiClient call (v2 prod calls = source of truth per project memory).

## 8. Model/effort/subagent strategy
- Discovery: DONE — 2 parallel Haiku Explore agents (read-only).
- Architecture review: Fable 5 main thread, high effort — DONE (this note).
- Implementation: Sonnet subagents, medium effort, one per phase/page-cluster; main thread reviews diffs.
- Verification: Fable 5 main thread, high effort — `tsc --noEmit`, `npm run lint`, `npm run build`, targeted vitest, per phase.
- Documentation: inline by main thread (cheap, no agent overhead).

## 9. Structural issues found (need decisions)
1. **Data layer mismatch** — v2 server actions hit upstream directly; violates v3 BFF rule. Must rewrite all ~40 calls as apiClient + React Query. No alternative. (Decision: forced, informational.)
2. **7 contexts don't exist in v3** — replace: profile-context→auth-slice, toast-provider→sonner, top-loader-router→`@/i18n/navigation` router, auth-context.danVerified→auth-slice user, payment/company contexts→React Query queries in feature. No new contexts.
3. **439-line user/organization monolith** — decompose; also partially redundant with v3's existing profile dropdown/dialog. Options: (a) migrate as page anyway (parity), (b) slim to XYP-sync + org overview only.
4. **Double sidebar UX** — v2 nests its own profile sidebar inside app sidebar layout. Options: (a) keep nested sidebar (parity), (b) tabs/segment nav v3-style, (c) fold into app sidebar. 
5. **Possibly dead pages** — `user/payment` (8L stub), `user/settings` (157L), `api-connection` (subscription-gated). Drop or keep?
6. **No form library in v2** — manual state + server validation. v3 convention = RHF+zod. Rewrite modal forms accordingly (behavior-preserving).
7. **Hardcoded route strings** in v2 org page → constants in feature config.

## 10. Decisions (approved 2026-07-03)
- [x] Phase order A→G approved; implement phase by phase, verify each before next.
- [x] Issue 3 (user/organization monolith): option (b) — slim down; reuse v3 profile switching; keep XYP-sync + org overview only; pause and ask on unclear/risky parts.
- [x] Issue 4 (double sidebar): option (b) — tabs/segments inside profile area; reusable structure for future settings-like pages. **Revised 2026-07-06 (`ddbb048`)**: tab bar replaced by vertical grouped second sidebar (w-56) — see §11 Post-G2 log. Mobile fallback undecided (§16).
- [x] Issue 5 (dead-ish pages): deferred — `user/payment` (stub), `user/settings`, `api-connection`. No placeholder routes created because the new tab nav only renders migrated (enabled) pages → no broken links exist.
- [x] Vault note stays flat at this path.

## 11. Implementation log

### Phase A — shell (2026-07-03) ✅
Executor deviation from plan: implemented inline by main thread (Fable 5) instead of Sonnet subagent — 7 small files, briefing cost > work cost. Subagents from Phase B.

Discovery bonus: v3 `src/i18n/languages-data/{mn,en,cn,kr}.json` ALREADY contain the full `profileDropdown` namespace (inherited from v2) — zero translation additions needed for nav; verified all 13 needed keys present in all 4 locales.

Built:
- `src/features/profile/config/nav.ts` — `ProfileNavItem` type + `USER_NAV`/`ORG_NAV` (all v2 menu entries; `enabled` flag = migrated; deferred pages listed as comments, no entries).
- `src/features/profile/components/ProfileNav.tsx` — client tab nav (segmented-control style matching v3 `tabs.tsx` look), profile-type-aware ordering (CITIZEN → user menus first, ORG → org menus first; both groups merged in one row, mirroring v2's MAIN/SECOND menu access), renders only `enabled` items. Reads `selectedProfile` from Redux auth-slice; `Link`/`usePathname` from `@/i18n/navigation`.
- `src/features/profile/components/PersonalInformation.tsx` — placeholder (Phase B target), sets breadcrumb via `useSetBreadcrumb` (Тохиргоо → Хувийн мэдээлэл).
- `src/features/profile/index.ts` — public surface.
- Route files: `profile/layout.tsx` (nav + children), `profile/page.tsx` (async redirect → `/profile/user/personal-information`, mirrors contract/create pattern), `profile/user/personal-information/page.tsx` (thin page).

Not carried over from v2 shell (intentional):
- profile-header subscription banner — depends on payment context, deferred to subscription phase (E/G).
- nested 260px profile sidebar + mobile dropdown — replaced by tab nav per approved decision 4(b).
- v2 loading.jsx — not needed yet (placeholder content instant).
- CITIZEN-block gate on org routes — deferred to Phase G (no org routes exist yet; nav never links there).

### Phase B — personal-information (started 2026-07-03)

Recon (haiku agent, v2 deep-read) key facts:
- v3 `User` type + auth-slice already hold ALL display fields (names, email+verified, danVerified, address codes+names, mobile, stamp/profile/signature image urls) — display is free, no new GET for page body.
- Refresh-after-mutation pattern: `router.refresh()` → server `getAuthData` → `AuthInitializer` dispatch.
- BFF gaps found: (1) no `auth-v2` proxy (phone verify endpoints live on `AUTH_URL_V2`) — adding `src/app/backend/auth-v2/[...path]/route.ts` mirroring auth catch-all with `getAuthUrlV2()`; (2) generic proxy `req.json()`s the body → drops multipart — adding dedicated `backend/auth/user/upload-user-image/route.ts` (static beats catch-all), mirrors `file/upload` route.
- Endpoints extracted (v2 → v3 browser path): provinces `GET /auth/location`; cascade `GET /auth/location/list?parentCode=`; countries `GET /public-api/v1/misc/countries`; address `POST /auth/user/update-address`; SMS send `POST /auth-v2/user/profile/verify-mobile`; SMS confirm `POST /auth-v2/user/profile/confirm-mobile-verification`; image `POST /auth/user/upload-user-image` (FormData, jpg/jpeg, 5MB).

**Scope split (risky parts held back per approved working rule):**
- B1 (implementing now, Sonnet subagent): section cards (own info, email, phone, address, profile image, stamp display+preview), phone-change dialog (SMS OTP full flow), address-change dialog (cascading selects), profile image upload, BFF routes above. Deferred controls rendered disabled ("Тун удахгүй").
- B2 (needs user approval before implementation): EmailChangeModal (contains ACCOUNT-MERGE flow via `/user/check-merge-account` + `/user/merge-account` — auth-critical, v2 UX not fully extracted), DAN socket linking (`handleLinkDan`, DAN_SOCKET_URL flow), StampCreateModal (not inspected in v2), VerificationHeader + VerificationStepsModal (depend on the above actions), SignatureCreateModal.

### Phase B1 — implemented + verified + committed `63ed860` (2026-07-03)

Sonnet subagent implemented (first attempt died on session limit; resumed agent stalled → killed, fresh agent completed). Main thread (Fable) reviewed every file + re-ran all verification independently.

Built (18 files, +1,345):
- `features/profile/api/client.ts` — profileApi: location, countries, update-address, verify/confirm mobile, upload image. **Payload corrections found against spec during v2 re-read**: verify-mobile = `{mobileNumber, countryCode}`; confirm = `{code}` (not `{verifyCode}`) — from v2 profile-context.jsx.
- `features/profile/api/queries.ts` — profileKeys + useProvinces/useLocationChildren/useCountries (staleTime Infinity, enabled-gated) + 4 mutations. Mutations don't invalidate — user lives in Redux; success → `router.refresh()`.
- `features/profile/schema/index.ts` — makePhoneChangeSchema (stage-as-form-value pattern for conditional validation), addressChangeSchema (no rules — v2 parity, documented).
- `components/personal-info/` — 6 section cards + PhoneChangeDialog (send SMS → countdown from response expiryDate via differenceInSeconds → 6-slot input-otp → confirm) + AddressChangeDialog (cascade + child-reset + numeric quarter sort). Dialogs mount fresh per open (v2 parity).
- BFF: `backend/auth-v2/[...path]/route.ts` (getAuthUrlV2), `backend/auth/user/upload-user-image/route.ts` (static beats catch-all; multipart forward).
- Locales: +3 stamp keys × 4 files.

Behavior changed intentionally: 5MB oversize now BLOCKS (v2 toasted but uploaded anyway — bug not reproduced); shadcn Select replaces react-select; input-otp replaces plain input (6 digits = assumption, ponytail-commented); address names read from user (v3 provides addressProvinceName etc; v2 did client-side lookup).

Verification (main thread re-ran, 2026-07-03):
| Check | Result |
|---|---|
| `npx tsc --noEmit` | PASS exit 0 |
| eslint (profile + backend + route) | PASS exit 0 |
| `npm run build` | PASS exit 0 (pre-existing sidebar.* cn/kr MISSING_MESSAGE warns only) |
| `npx vitest run` | PASS 28/28 |
| Translation audit (script: extracted every t() call, 59 keys, 4 namespaces × 4 locales) | none missing |
| Built routes | personal-information page + auth-v2 + upload routes emitted |

Assumptions (labeled in code with ponytail comments): countries prefix `/public-api/v1/misc/countries` unverified vs live backend; upload response typed string; VerifyMobileResult={expiryDate?}; ~~OTP length 6~~ **FALSIFIED — real SMS code is 4 digits, fixed in bf4e7b6 (2026-07-06) along with MN country default**. First live API hit will surface mismatches — manual smoke needed (SMS flow at least partially live-validated by the OTP fix).

Manual smoke checklist (not yet done): image upload round-trip, SMS send+countdown+confirm with real number, address save + refreshed names, stamp preview, disabled B2 buttons show "Тун удахгүй", locale switch (en/cn/kr) renders all labels.

### Phase B2 — email connect/change/merge, committed `d8450b9` (2026-07-03)

User approval: B2 = email change+merge ONLY; DAN linking, stamp create, verification header remain deferred/disabled. Implemented inline by main thread (auth-critical, high effort).

Recon (haiku, v2 EmailChangeModal deep-read) key facts:
- v2 flow: check-merge-account {username} → {registered, danVerified}; merge offered when registered && !danVerified; blocked (toast userMail/newMailUser) when registered && danVerified; else send-email-confirmation {email[, registered:true]} → 6-digit OTP → verify-email-confirmation {otp} OR merge-account {username, otpCode}.
- **CRITICAL: session rotates on success** — response {token, credentials{id}}; v2 setToken (access cookie) + setCookie('profile', id), no logout/redirect. Refresh token untouched.
- Duplicate-send detection = literal string "И-мэйл илгээсэн байна." (kept, commented).
- EMAIL_REG copied verbatim. Error namespaces split: email→changeMail, OTP→actions.
- All changeMail/actions keys already in all 4 v3 locales — zero i18n additions.

Built (8 files, +513):
- `backend/auth/user/_shared/rotate-session.ts` — shared session-rotating POST handler: forwards upstream, on success sets access_token + selected_profile_id httpOnly cookies (full setTokens if response carries refreshToken); strips token from browser response.
- `backend/auth/user/{verify-email-confirmation,merge-account}/route.ts` — static routes beating auth catch-all.
- Feature: client.ts +4 endpoints, queries.ts +4 mutations, schema makeEmailChangeSchema (stage pattern), EmailChangeDialog (state machine: input → merge-offer? → OTP → confirm/merge → router.refresh()), EmailSection enabled (create/update modes).

Verification (main thread, 2026-07-03): tsc PASS exit 0; eslint PASS exit 0; build PASS exit 0 (pre-existing sidebar.* warns only); both routes emitted in .next; vitest 28/28; ICU {email} param confirmed in addressVerMail.

Assumptions: credentials field id vs userId unknown → reads both; refresh-token validity after merge unverified (v2 had same gap — if backend invalidates it, next silent refresh logs user out); duplicate-sent string may arrive as success data or error message → both paths route to OTP stage.

Manual smoke needed (auth-critical): real email connect round-trip; merge flow with second account; session continuity after confirm (no logout, user data refreshed); update-mode change for verified email.

### Phase C — security (started 2026-07-03)

Recon (haiku, v2 security cluster) key facts:
- **No session rotation in any flow** — 2FA/password/anti-phishing/advanced-config are plain authenticated calls; no new cookie-setting BFF routes needed.
- Endpoints: 2FA init/activate `POST /user/createTwoFA[?code=]` (init returns QR url + secret key), deactivate `POST /user/removeTwoFA?code=`; password `POST /user/change-password {newPassword, oldPassword}`; anti-phishing CRUD on `/user-security/anti-phishing` (GET/POST/PUT/PATCH toggle/DELETE — all AUTH_URL); advanced configs on BACKEND_URL `/user-advanced-config` (GET + POST update-method-value).
- **Infra gap**: v3 auth catch-all maps PUT/PATCH/DELETE → okNoop (no-op!). Anti-phishing needs them → catch-all gets real handle() for those methods (the one shared-infra change of this phase).
- v2 smells: 2FA secret regenerated on every page load (server-side POST in page body) → v3 fetches lazily on modal open; PasswordValidator was display-only, PASSWORD_REG never enforced client-side → v3 enforces in zod (server still validates).
- Deps: @mantine/hooks useClipboard → replaced with navigator.clipboard; QR is server-rendered image URL (no QR lib needed).
- AntiPhishingModal type="show" never triggered in v2 → skipped.

Implementation: Sonnet subagent (first attempt died on session limit → fresh relaunch, no resume). Committed `c54461c` (22 files, +1,397, 2026-07-04).

Built: components/security/ (SecurityContent + VerificationMethods/Password/ActionSecurity sections + TwoFa/TwoFaDeactive/PasswordChange/AntiPhishing/AntiPhishingDeactive modals + PasswordValidator + StatusBadge), 11 endpoints in client.ts, 3 queries + 8 mutations in queries.ts (anti-phishing/config mutations invalidate their keys), password/2FA/anti-phishing schemas with v2 regexes verbatim, security route page, nav security enabled, auth catch-all PUT/PATCH/DELETE → real handle().

Preserved: anti-phishing conditional two-step create (toggle→create only when no prior record — v2 `!!data` quirk), toggle→DELETE deactivate order, method availability gating, OTP flows. Changed (commented): lazy 2FA secret fetch (v2 regenerated per page load), client-side PASSWORD_REG. Skipped: v2 dead no-op email-connect button on security page (real flow lives in EmailSection), AntiPhishingModal type="show" (never triggered). Reused PhoneChangeDialog for phone row (intra-feature).

Verification (main thread, 2026-07-04): tsc PASS; eslint PASS; build PASS exit 0 — confirmed ALL MISSING_MESSAGE warns are pre-existing sidebar.* cn/kr, none new; vitest 28/28; scripted i18n audit 137 keys × 7 namespaces × 4 locales — none missing (+2 security keys added all locales); ICU {count} param confirmed.

Assumptions: AdvancedConfig response shape typed from v2 usage (ponytail-commented); getAntiPhishing 404-vs-empty behavior unknown → create-step trigger condition inherited from v2 as-is.

Manual smoke needed: 2FA activate/deactivate with real authenticator, password change, anti-phishing round-trip, config toggles, QR image renders through BFF.

### Phase D — signature: DEFERRED (user decision, 2026-07-04)

Recon done (haiku). Findings that triggered deferral:
- v2 flow needs libs absent from v3: `react-signature-canvas` (draw), `react-easy-crop` (crop/zoom/rotate), `heic2any` (HEIC), `swiper` (embla would replace).
- Background-remove calls a separate image-processing service (`/image-processing-api/bg-remove-v2`) proxied at nginx level in v2 — v3 has no route/env for it (confirmed: only a middleware matcher exclusion in v2, no app route).
- Endpoints (for future implementation): upload `POST BACKEND /file/upload` FormData {file, entity:"signature"} → {fileUrl}; create `POST BACKEND_V2 /signature/create` {imgUrl, format:"STANDARD"|"SQUARE"}; history `GET BACKEND /signature/history-list?sortParam=createdDate&currentPage=&pageSize=`; bg-remove `POST BASE_URL/image-processing-api/bg-remove-v2` FormData {file, color:"#000000"}.
- Flow spec: type select (draw/upload) → draw canvas (3:1 or 1:1, undo history) or file upload (jpg/png/webp, HEIC convert) → crop (zoom 0.2–5x, rotate ±45°, optional bg-remove) → upload + create → getUserProfile refresh.

User decisions: no new libraries now; HEIC later; bg-remove skipped. Signature nav tab stays `enabled: false` ("Тун удахгүй"). Revisit as its own task when lib decision changes. For that revisit: v2 env has `BG_REMOVER_URL=https://image-processor-api.e-geree.mn` — the image-processing service base URL for bg-remove.

### Phase E — table pages (started 2026-07-04)

Recon (haiku) key facts:
- login-history `GET AUTH /user/login-history` (5 cols: IP/browser/OS/device/date, icon mappings); payment-history `GET PAYMENT_URL /payment/history` (4 cols + VAT/receipt modal when vatStatus=COMPLETED && lotteryNumber); subscription `GET BACKEND /subscription-request/list` (6 cols, status badges, search); referral: detail card + usage list + create/update code on BACKEND `/user-referral-code/*`.
- **New upstream**: PAYMENT_URL absent from v3 → adding env (`https://payment-api.e-geree.mn/payment-api/v1` from v2 env), `getPaymentUrl()` helper, `/backend/payment/[...path]` BFF catch-all.
- v2 smells: pagination `current` vs `currentPage` ambiguity (referral); subscription date formatted differently from other pages (kept, commented); `checkReferralCode` action dead code (not migrated).
- Implementation: Sonnet subagent dispatched; table approach (shared data-table vs plain shadcn Table) decided by agent after inspecting coupling, one approach for all 4 pages.

Committed `b6966cc` (23 files, +1,201, 2026-07-04). Agent chose shared DataTable (generic, manualPagination supported, toolbar optional — matches ReceivedContent pattern). VAT/receipt modal DEFERRED (needs react-qr-code — no-new-libs rule; disabled Тун удахгүй button when vatStatus=COMPLETED && lotteryNumber). Icons: v2 PNG assets → react-icons brand icons (commented). Pagination URL params → component state + placeholderData (commented). checkReferralCode dead code not migrated.

**DEPLOY NOTE: `.env*` gitignored — PAYMENT_URL (`https://payment-api.e-geree.mn/payment-api/v1`) must be added to deploy environments out-of-band.**

Verification (main thread, 2026-07-04): tsc PASS; eslint PASS; build exit 0 (only pre-existing sidebar.* warns); vitest 28/28; all 6 profile subpages + payment proxy emitted in .next; i18n audit 150 keys × 11 namespaces × 4 locales — none missing (0 additions needed).

Manual smoke needed: each table loads + paginates against live APIs; referral create/edit/copy round-trip; payment-history against payment-api (first ever v3 call to that upstream); receipt button renders disabled for VAT rows.

### Phase F — user/organization slim (DONE, commit 819d2c1, 2026-07-04)

Implemented by fresh Sonnet subagent per spec below. Files: route page `profile/user/organization/page.tsx`; `components/organization/{OrganizationContent,OrganizationCard}.tsx`; `hooks/useOrgLayout.ts` (companyLayout localStorage, SSR-safe render-phase hydration); client.ts +3 fns (getCompanyEmployees, getLegalEntitySyncStatus, syncLegalEntity); queries.ts +3 hooks (sync mutation invalidates org list + sync status, no polling); nav organization enabled:true; index.ts export. Reused `Company`/`CompanyEmployee` from `@/types/company-types`; profile-switch payload mapped same as `formatProfileList` in `src/shared/lib/custom-utils.ts` so useProfileSwitch unmodified. Two NEW i18n keys added to all 4 locales: `security.syncingOrgKHUR` (sync tooltip), `profileDropdown.noOrganization` (empty state). Switch button label reuses `actions.choose` (no dedicated key). Verify: tsc PASS, lint PASS (5 pre-existing warnings elsewhere), build PASS (known cn MISSING_MESSAGE noise), vitest 28/28. Not verified: manual browser smoke (grid/list toggle, search, sync flow, profile switch from card).

Original recon spec:

Recon (haiku, 2026-07-04). Approved slim-down (decision 2b): keep org overview + XYP sync; drop profile-switch duplication.

KEEP: grid/list toggle (localStorage "companyLayout", default grid), client-side search (name lowercase includes OR regNum includes), card fields (logo Avatar, name, danVerified badge RiVerifiedBadgeFill, regNum with security.rNumber, contract stats total/received/sent shown when verified = categoryIds.length>0 && stampFileUrl), XYP sync button (security.downloadOrgKHUR, disabled+tooltip while syncStatus PENDING/RUNNING, gate on user.danVerified → personalInfo.pleaseLinkDan warning), empty state.

ENDPOINTS: org list `GET /public-api/v1/company-employee` (CompanyEmployee[], v3 already calls it server-side in getAuthData); sync status `GET /public-api/v1/legal-entity/current` → {syncStatus: PENDING|RUNNING|SUCCESS|FAILED}; sync `POST /public-api/v1/legal-entity/sync` (no body). No polling in v2 — single call + manual refetch.

DROP: handleChangeProfile/switchProfile duplication (use existing src/hooks/useProfileSwitch, mirror profile-dropdown-content.tsx usage), per-card ORG_MENUS dropdown (org pages = Phase G), VerificationAction component + its switchProfile (step targets = Phase G features), list-view 3-dot menu.

Card action in v3: single "switch profile" button via useProfileSwitch, hidden/disabled when already selected. Nav tab: flip organization enabled:true.

i18n keys needed (verify, likely present): profileDropdown.searchOrganization/organization, security.downloadOrgKHUR/rNumber, status.receivedDocument/sentDocument, personalInfo.pleaseLinkDan, payment.soon. v2 sync tooltip was hardcoded Mongolian ("ХУР-аас байгууллагын мэдээлэл татаж байна...").

Two implementation attempts died on session limits before writing anything (tree stayed clean). Next session: relaunch Sonnet subagent fresh with this spec.

### Phase G — scope split (user decision, 2026-07-04)

G split into 3 sub-phases, one per chat session, stop + hand off between each: **G1 org-info** (✅ below), **G2 teams + user-manage** (~2,620L), **G3 dashboard + subscription** (chart lib: v3 has recharts — confirm). api-connection stays deferred (user decision). Per-sub-phase method unchanged: haiku Explore recon → sonnet subagent implements → Fable main thread reviews + verifies → commit → vault update.

### Phase G1 — organization/information (DONE, commit 9c9291f, 2026-07-04)

Recon report (haiku): scratchpad org-info-report.md — v2 cluster 8 components 1,379L + 5 action files. First Sonnet impl agent killed by session limit mid-task (had finished client.ts/queries.ts/schema/3 locales); relaunched fresh per playbook — second agent verified partial work, fixed 2 real bugs in it (getCompanyRoles response unwrap `{companyApplicationRoles:[...]}`; `Company.externalSystem` typed `{apiKey?: string|null}|null`), completed the rest.

Files: route `profile/organization/information/page.tsx` (thin); `components/organization/{OrgInfoContent,LogoSection,OrgInfoSection,CategoryChangeModal,StampDisplaySection,ApiKeySection,EditIpListModal,ApiPdfDownloadSection,OrgDeleteSection}.tsx` (~1,150L); client.ts +9 fns; queries.ts +9 hooks (key factories, invalidation on mutations); schema +company-integration/ip-list zod; nav orgInformation enabled:true. BFF change: `public-api/v1/[...path]/route.ts` PUT/PATCH/DELETE upgraded okNoop → real proxy (auth proxy precedent; needed by company-integration PUT + company/remove DELETE).

Endpoints: company/update POST, company/remove DELETE (no body — session-scoped, v2 parity), company/ip-list GET + update-ip-list POST, company-integration GET/PUT/POST, auth/fetch-company-roles GET (double "auth" segment correct), logo upload reuses existing `/backend/file/upload`.

Gates: CITIZEN → redirect to /profile/user/organization; ROLE_COMPANY_CHANGE_CATEGORY gates category change; danVerified hides delete section entirely (hard gate).

Deviations (intentional): stamp create/change omitted (deferred, react-easy-crop); Mantine useForm → RHF+Zod, useClipboard → navigator.clipboard+sonner; v2 5MB logo bug fixed (blocked, not toast-then-upload); category icons plain text (no DynamicSvgIcon in v3); delete confirm = shadcn AlertDialog inline (v2's ActionVerificationModal is plain confirm, no OTP gap). Main-thread review fix: OrgDeleteSection switches via real personal profile from `s.auth.profileList` (find CITIZEN) instead of fabricated `{value:userId,title:""}` object.

i18n: 9 new orgInfo keys added to all 4 locales (first agent missed mn.json — second added; scripted audit: orgInfo 23 keys identical × 4; remaining locale gaps are pre-existing sidebar/contractCreate cn/kr + profileDropdown.apiConnection, out of scope).

Verify: tsc PASS, lint PASS (5 pre-existing warnings elsewhere), build PASS (known cn MISSING_MESSAGE noise), vitest 28/28. Not done: manual browser smoke (also still pending for Phase F — user chose to skip before G).

### Phase G2 — teams + user-manage (DONE, commit 566b7ca, 2026-07-04)

Recon: haiku Explore (scratchpad g2-recon-report.md) — report had 3 payload errors, main thread re-verified EVERY endpoint against v2 source and wrote corrected spec (scratchpad g2-implementation-spec.md). Corrections: team remove-employees payload = `{teamId, teamEmployeeIdList:[teamMemberRowId]}` (not employeeIdList); remove-employee/update-role use `companyEmployeeId` (not employeeId); set-view-all-contracts uses `employeeId` (v2 inconsistency kept); checkRelatedInfo response `data===true` ⇒ needs delegate transfer (not `{canDelete}`).

Impl: first Sonnet agent killed by session limit (had done api/client+queries+schema+types+icon libs); relaunched fresh per playbook — second agent audited partial work, found 1 real bug (GenIcon called per-render in TeamIcon → react-hooks/static-components error; fixed with module-load icon cache + createElement), completed rest.

Endpoints added (all V1 unless noted): team/list, team/{id}, team/create POST, team/update PUT, team/{id} DELETE, team/add-employees, team/remove-employees, company-employee/employee-list (paged+search), add-employee {usernameList}, remove-employee, update-role, set-view-all-contracts, check-related-info; contract-label (V2, one-liner duplicated — boundaries forbids documents-feature import); subscription GET (current org sub, first use in v3).

Files (29, +3,383): components/teams/{TeamsContent,TeamDetailContent,TeamMembersList,CreateTeamDialog,EditTeamDialog,AddMemberDialog,EmployeeSelector,TeamIcon}, components/user-manage/{UserManageContent,MemberList,EmployeeAddDialog,ContractTransferDialog,SubscriptionPlanCard}, lib/{team-colors,team-icons,team-icons-data} (20 duotone icons extracted from v2 icon_data.js 206K, path data diffed 0 mismatch; GenIcon from react-icons), 3 route pages (teams, teams/[slug] Next-15 async params, user-manage), client.ts +14 fns, queries.ts +key factories +hooks (employeePage placeholderData + enabled param; relatedInfo fetchQuery staleTime Infinity = v2 session cache parity), schema team create(2–40)/edit(2–20)/desc-10–255-optional/email-or-registry rows, CompanyEmployee.user typed, nav teams+userManage enabled.

Gates (v2-verbatim): create-team + team-member-delete = role ADMIN|OWNER; role select disabled for OWNER-row/self/MEMBER-viewer/!ROLE_COMPANY_CHANGE_EMPOYEE_ROLE (sic backend typo); actions column = role!==MEMBER && ROLE_COMPANY_REMOVE_USER; viewAllContracts column OWNER-only; add-employee btn = sub ACTIVE && ROLE_COMPANY_ADD_USER && seats free; CITIZEN guard redirects both pages.

Deviations (ponytail-commented): AddMemberDialog submit label fixed (v2 used createTeam key); v2 handleChangeRole undefined-`error` bug fixed; pagination/search = component state + debounce (Phase E precedent, not URL params); useFieldArray replaces v2 unregister trick; SubscriptionPlanCard "get subscription" disabled + payment.soon tooltip (payment modal = G3-scope); EmployeeAddModal character illustration skipped (asset absent); react-select label multi-select → toggle-chips (CategoryChangeModal pattern); ContractTransferDialog keeps v2 pageSize=100 TODO.

i18n: 81 keys used, scripted audit × 4 locales = 0 missing. 2 NEW keys added ×4: teams.selectLabel ("Хавтас сонгох" was hardcoded), massReport.transferMemberWarning (hardcoded Alert text).

Verify (main thread re-ran): tsc PASS exit 0; eslint profile+app PASS 0 issues; build exit 0 (all 560 MISSING_MESSAGE = pre-existing sidebar.* cn/kr, 0 new; 3 routes emitted); vitest 28/28. Not done: manual browser smoke (F/G1/G2 all pending user decision).

### Post-G2 unplanned work (2026-07-06, logged retroactively by 2026-07-07 audit)

Six commits shipped outside the phase plan and were not recorded here at the time:
- `b05d916` fix(auth): `useHydrated()` guard added to 15 profile components branching render on redux auth state (hydration mismatch fix). **Now a required pattern — playbook step 13.**
- `ddbb048` feat(profile): ProfileNav redesigned — horizontal segmented tab bar → vertical grouped second sidebar (w-56); breadcrumb polish; profile quick-links in profile-dropdown-content. **Revises decision 4(b)** (noted there).
- `52da1d7` style(ui): rounded-lg applied across 14 profile files.
- `bf4e7b6` feat(profile): dialog loading states; **OTP_LENGTH 6→4** (B1 assumption falsified — real SMS code is 4 digits); phone country default MN.
- `c4fcb22` i18n: `profileDropdown.title`/`orgMenu` + `security.loading` keys, all 4 locales.
- `1cb7d81` fix(auth): logout route now clears auth cookies (previously never cleared; adjacent to B2 rotate-session cookie work — §18).

Related in documents feature (2026-07-07): `1ac18a2` GSIGN flow (2d-3), `ff09d0b` contract payment flow (2d-4) — full payment machinery (method-list → initiate → QPay QR → socket status) now lives in `features/documents` (`ContractPaymentModal`). Directly relevant to G3 — see pre-flight below.

### G3 pre-flight (gap-audit findings, 2026-07-07 — read before starting the G3 chat)

1. **Payment-modal reuse decision — BLOCKS G3.** Payment machinery is complete in `features/documents` (ContractPaymentModal ~276L, typed client fns; `socket.io-client` 4.8.3 installed). eslint-boundaries forbids profile→documents imports (G2 already hit this and duplicated a one-liner — this is ~250+ lines of money-path logic). DECIDE: hoist to `shared/` vs duplicate into profile. Undecided = G3 stalls mid-chat.
2. **Charts**: v2 dashboard uses chart.js + react-chartjs-2 (DoughnutChart with min-count-3 display trick + original-count tooltips) and react-tailwindcss-datepicker — none exist in v3. Use **recharts 2.15.4** (present, "confirm" resolved) + **react-day-picker 9.14** for the date range (no-new-libs rule; no shared range-picker component exists yet — build one).
3. **Static assets**: 7 dashboard icons (tree-with-small/water-drop/co2/trash-with-small .png + newspaper-blue/send-blue/inbox-filled-blue .svg) copied v2→v3 `public/assets/photos/icons/` on 2026-07-07 ✅.
4. **Subscription attach points G3 must own** (beyond the two org pages): (a) profile-header subscription banner — orphaned Phase A deferral; (b) `user/subscription` SubscriptionPlan card (current plan + buy button) — silently dropped in Phase E, restore for parity; (c) enable G2's disabled "get subscription" button in user-manage SubscriptionPlanCard.
5. **OrgVerificationHeader** — unowned v2 feature (org layout banner: verification progress + next-step → CategoryChangeModal/StampCreateModal). Decide migrate-in-G3 vs explicit defer BEFORE G3 scope freeze (§16).
6. **`useHydrated()` guard mandatory** for any new component branching render on `s.auth` (playbook step 13).
7. **org/subscription history table ≈ Phase E user table** (same `/subscription-request/list` endpoint) — reuse E's components/columns, don't rebuild.

## 12. Files changed (Phase A — all new, nothing modified)
```
src/features/profile/config/nav.ts
src/features/profile/components/ProfileNav.tsx
src/features/profile/components/PersonalInformation.tsx
src/features/profile/index.ts
src/app/[locale]/(protected)/(with-sidebar)/profile/layout.tsx
src/app/[locale]/(protected)/(with-sidebar)/profile/page.tsx
src/app/[locale]/(protected)/(with-sidebar)/profile/user/personal-information/page.tsx
```
Committed 2026-07-03 (Phase A). Note: `ddbb048` (2026-07-06) later rewrote ProfileNav; `nav.ts` gained `buildNavGroups` + `nav.test.ts` on 2026-07-07.

## 13. Verification results

### Phase A (2026-07-03, all commands actually run)
| Check | Command | Result |
|---|---|---|
| TypeScript | `npx tsc --noEmit` | PASS — "No errors found", exit 0 |
| Lint | `npx eslint src/features/profile "src/app/.../profile"` | PASS — "No issues found", exit 0 (boundaries + check-file rules included) |
| Build | `npm run build` | PASS — exit 0. Pre-existing warnings: `MISSING_MESSAGE sidebar.templateDesc/openDocumentDesc/linkedTemplateDesc (cn)` from `CreateDocumentDialog.tsx` — NOT ours, untouched file, left alone |
| Route emission | ls `.next/server/app/.../profile` | PASS — profile page + user/personal-information emitted |
| Translation keys | python check, 13 keys × 4 locales | PASS — none missing |
| Unit tests | `npx vitest run` | PASS — 28/28, 0 fail |

Not verified (needs manual browser smoke): tab renders with correct label/icon; active state on personal-information; `/profile` redirect in running app; breadcrumb shows Тохиргоо → Хувийн мэдээлэл; profile-type ordering when switching to org profile. Static + build verification only for these.

Assumptions: `profileDropdown.settings` is the right section label for breadcrumb root (v2 profile-header used it as the section title).

## 14. Unresolved issues (reconciled 2026-07-07)
- ~~v2 upstream endpoint URLs not yet extracted per action~~ **RESOLVED** — extraction done per phase through G2.
- ~~Chart library for org dashboard~~ **RESOLVED** — recharts 2.15.4 in v3 package.json. Caveat: v2 actually uses chart.js + react-chartjs-2 + react-tailwindcss-datepicker (all absent) → G3 rebuilds with recharts + react-day-picker (see G3 pre-flight §11).
- ~~Whether v3 BFF proxy covers all needed upstream paths~~ **RESOLVED** — C upgraded auth catch-all PUT/PATCH/DELETE, E added payment catch-all, G1 upgraded public-api/v1 methods. Consolidated deploy/security ledger: §18.
- Response types unknown (v2 untyped) — ongoing per phase; derive from usage, label assumptions.
- Pre-existing cn-locale MISSING_MESSAGE warnings in build (CreateDocumentDialog) — out of scope, flagged to user.
- ~~No unit test for ProfileNav ordering/filter logic~~ **RESOLVED 2026-07-07** — grouping/filter extracted to `buildNavGroups` in `config/nav.ts` + `nav.test.ts` (ordering per profile type, enabled-filter, path uniqueness); vitest/tsc/eslint all pass.

## 15. Reusable migration checklist (playbook v1 — refine after Phase A/B)
1. Inventory FULL dependency iceberg: route files + `components/pages/<x>` + layout components + actions/services + contexts + translations. Route lines lie; measure everything.
2. Map every data call: server action/fetch → upstream URL → v3 apiClient(`/backend/...`) + React Query hook with key factory.
3. Map contexts/state → v3 equivalents (auth-slice, sonner, i18n navigation, React Query) before writing components.
4. Target shape: `src/features/<x>/{api,components,hooks,lib,schema,types,index.ts}` + thin route files in `app/[locale]/...`; respect boundaries lint (no cross-feature internals).
5. Naming: components PascalCase, hooks `use*` camelCase, rest kebab-case.
6. Forms → RHF + Zod v4, error messages as i18n keys.
7. Translations: extract used keys per namespace, add to ALL 4 locale files.
8. Nav/links via `@/i18n/navigation`; breadcrumb via `useSetBreadcrumb`.
9. Auth/permission gates: recreate from auth-slice (profile type, verification flags) — never trust context names, read the actual gate logic.
10. Verify per phase: `tsc --noEmit` → `npm run lint` → `npm run build` → targeted vitest → manual smoke list. Record pass/fail + errors verbatim.
11. Flag structural smells before fixing; get approval; log decision here.
12. Update this note + worklog as you go, not at the end.
13. Any client component whose render branches on redux auth state (`user`, `selectedProfile`, `profileList`) must gate with `useHydrated()` (precedent: `b05d916` retrofitted 15 components) — otherwise SSR hydration mismatch.

## 16. Deferred / unowned ledger (consolidated 2026-07-07)

Every deferral in one place. ❌ = fell out of tracking (2026-07-07 audit find), needs decision; ✅ = recorded user decision with owner/trigger.

| Item | Origin | Owner / trigger | Tracked? |
|---|---|---|---|
| OrgVerificationHeader (org layout verification banner + provider, drives CategoryChangeModal/StampCreateModal) | never phase-inventoried | **NONE — decide before G3** | ❌ |
| profile-header subscription banner (remaining-days + buy btn) | Phase A deferral "to E/G" | **orphaned → assigned to G3 pre-flight #4a** | ❌→G3 |
| user/subscription SubscriptionPlan card (current plan + buy) | silently dropped in E | **assigned to G3 pre-flight #4b** | ❌→G3 |
| G2 user-manage SubscriptionPlanCard disabled buy button | G2 | G3 (payment modal), pre-flight #4c | ✅ |
| Payment-modal reuse decision (hoist to shared/ vs duplicate) | G3 pre-flight #1 | **user decision — BLOCKS G3** | ❌ pending |
| ProfileNav mobile fallback (v2 mobile dropdown dropped; current nav fixed w-56, zero responsive classes) | Phase A / `ddbb048` | **NONE — needs UX decision (drawer? dropdown?)** | ❌ |
| Phase D signature — whole page, incl. SignatureCreateModal from B2 list | user decision 2026-07-04 | revisit on lib decision (react-signature-canvas, react-easy-crop, heic2any) | ✅ |
| bg-remove (BG_REMOVER_URL=https://image-processor-api.e-geree.mn) | Phase D | with Phase D revisit | ✅ |
| DAN linking (handleLinkDan, DAN_SOCKET_URL) | B2 | deferred, UI disabled | ✅ |
| StampCreateModal + stamp create/change | B2 / G1 | react-easy-crop lib decision | ✅ |
| VerificationHeader + VerificationStepsModal (user branch) | B2 | depends on DAN/stamp above | ✅ |
| VAT/receipt modal (needs react-qr-code) | E | lib decision | ✅ |
| user/payment (8L stub), user/settings (157L) | decision 5 | deferred, no routes exist | ✅ |
| organization/api-connection | user decision | deferred (subscription feature gate) | ✅ |
| HEIC upload support (heic2any) | Phase D | later | ✅ |
| EmployeeSelector буруу endpoint — `useCompanyEmployees` → `GET /company-employee` нь хэрэглэгчийн ӨӨРИЙН бүртгэлүүдийг буцаадаг (компанийн staff биш); зөв нь `/company-employee/employee-list` (`useEmployeePage`). ShareLabelDialog-д ижил bug `bb966e9`-д засагдсан; profile тал засагдаагүй (`src/features/profile/components/teams/EmployeeSelector.tsx:50`, `src/features/profile/api/client.ts:516`) | Worklog 2026-07-07 илрүүлэлт | G-phase follow-up | ✅ |

## 17. Manual smoke gate (consolidated — NONE done as of 2026-07-07)

All 8 phases shipped on static verification only (tsc/lint/build/vitest). Single gate below must pass before calling the migration production-ready. Auth-critical first.

- [ ] **B2 (auth-critical)**: email connect round-trip; merge flow with a second account; session continuity after confirm — open assumption: refresh-token validity after merge (silent-logout risk); update-mode change for verified email.
- [ ] **C (auth-critical)**: 2FA activate/deactivate with real authenticator (lockout risk); password change; anti-phishing CRUD round-trip; advanced config toggles; QR image renders through BFF.
- [ ] B1: profile image upload round-trip; SMS send+countdown+confirm (partially live-validated — OTP fixed to 4 digits, `bf4e7b6`); address save + refreshed names; stamp preview; disabled B2 buttons show "Тун удахгүй"; locale switch en/cn/kr.
- [ ] E: each of 4 tables loads + paginates live; referral create/edit/copy; payment-history — **first ever v3 call to payment upstream**; VAT receipt button disabled state.
- [ ] F: grid/list toggle; org search; XYP sync flow (danVerified gate + tooltip); profile switch from card.
- [ ] G1: org info edit; category change; IP list edit; api-key view/copy; logo upload; org delete (**destructive — test last, on a throwaway org**).
- [ ] G2: team CRUD + member add/remove; employee add (seat limit); role change gates; contract transfer; view-all-contracts toggle.
- [ ] Nav/shell (A + `ddbb048`): ordering per profile type; /profile redirect; breadcrumbs; **mobile viewport — expect breakage, no responsive fallback exists (§16)**.

## 18. Deploy / infra checklist (consolidated 2026-07-07)

Env (out-of-band — `.env*` gitignored):
- [ ] `PAYMENT_URL=https://payment-api.e-geree.mn/payment-api/v1` (Phase E)
- [ ] G3 likely: payment socket URL for subscription modal — check what `ContractPaymentModal` (documents 2d-4) already uses and reuse.

New/changed BFF surface — **security review recommended before prod**:
- [ ] `backend/auth-v2/[...path]` catch-all (B1) — new upstream AUTH_URL_V2
- [ ] `backend/auth/user/upload-user-image` (B1) — multipart forward
- [ ] `backend/auth/user/_shared/rotate-session` + `verify-email-confirmation` + `merge-account` (B2) — mints access_token/selected_profile_id httpOnly cookies, strips token from browser response
- [ ] auth catch-all PUT/PATCH/DELETE okNoop → real proxy (C) — widened for ALL auth paths, not just anti-phishing
- [ ] `public-api/v1` catch-all PUT/PATCH/DELETE okNoop → real proxy (G1) — same widening
- [ ] `backend/payment/[...path]` catch-all (E) — new upstream
- [ ] `backend/auth/logout` cookie-clearing fix (`1cb7d81`)
