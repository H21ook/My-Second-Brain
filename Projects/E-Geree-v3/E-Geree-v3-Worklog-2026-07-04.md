# 2026-07-04

## Profile migration — Phase C (security)
- Committed `c54461c` (22 files, +1,397). Sonnet subagent built (relaunch after session-limit death); Fable reviewed + verified: tsc/eslint/build/vitest PASS; i18n audit 137 keys × 4 locales clean; all build warns pre-existing sidebar.* only.
- Infra: auth catch-all BFF now proxies PUT/PATCH/DELETE (was okNoop) — anti-phishing needs them.
- Deviations (commented in code): lazy 2FA secret fetch; client-side PASSWORD_REG.
- See [[E-Geree-v3-Profile-Migration-Plan]] §11 Phase C.

## Profile migration — Phase D deferred + Phase E (table pages)
- Phase D (signature) DEFERRED by user: needs react-signature-canvas/react-easy-crop/heic2any (absent in v3) + bg-remove service route. Spec + BG_REMOVER_URL captured in plan note for revisit.
- Phase E committed `b6966cc` (23 files, +1,201): login-history, payment-history, subscription, referral on shared DataTable. New payment upstream: getPaymentUrl() + /backend/payment proxy. DEPLOY: PAYMENT_URL env needed out-of-band (.env gitignored).
- VAT modal deferred (react-qr-code). Verified: tsc/eslint/build/vitest PASS; i18n audit clean.

## Session stop (clean)
- Stopped mid-Phase-F by user. Tree clean at `b6966cc`; F recon written to plan note §11; F implementation not started (2 subagent attempts died on session limits, nothing on disk).
- Resume instruction: read [[E-Geree-v3-Profile-Migration-Plan]] → dispatch Phase F implementation per §11 spec → then Phase G (org branch, needs scope-split approval).

## Profile migration — Phase F (organization overview)
- Committed `819d2c1` (12 files, +434): /profile/user/organization page — org grid/list toggle (useOrgLayout, companyLayout localStorage), search, cards (danVerified badge, regNum, contract stats when verified), XYP sync (legal-entity current/sync endpoints, no polling), profile switch via existing useProfileSwitch. Nav tab enabled.
- Fresh Sonnet subagent per §11 spec (relaunch strategy worked). 2 new i18n keys × 4 locales: security.syncingOrgKHUR, profileDropdown.noOrganization.
- Verified: tsc/lint/build/vitest 28/28 PASS. NOT browser-smoke-tested.
- Session stop (user): F done, tree clean at `819d2c1`. Next chat: Phase G planning — needs user scope-split decision on 4.6k-line org branch first.
