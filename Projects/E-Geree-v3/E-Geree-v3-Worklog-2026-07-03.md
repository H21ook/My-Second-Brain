---
title: "Worklog: E-Geree v3 — 2026-07-03"
type: worklog
created: 2026-07-03
tags:
  - worklog
  - project/e-geree-v3
---

# Worklog — 2026-07-03

Continuation of [[E-Geree-v3-Contract-Detail-Page-Phase2d-Plan]]. Earlier today: Contract Detail Page UX refinement (breadcrumb slot, action dock merge, rail tabs, sticky fill header — commits through `58eb940`). This entry covers **2d-2c**.

## 2d-2c: OTP verify + submit finalize — commit `8dd7531`

**Scope:** wire real backend calls into `ActionButtons.tsx`'s `onApprove`/`onReview` (previously no-op placeholders), implementing the two-phase verify pattern (`verify:false` → OTP `methods` required → `verify:true`+`codes` → finalized).

**Findings before implementation:**
- No formal backend API spec exists in this repo (no Swagger/OpenAPI). Confirmed via direct read of `e-geree-v2/src/lib/actions/contract.js` that `approveContract`/`reviewContract` hit `${BACKEND_URL}/contract-action/approve` and `/review` — and v2's `BACKEND_URL` (`public-api.e-geree.mn/public-api/v1`) is the **same backend** v3 targets. This resolved the plan's "backend тулга" (needs confirmation) flag without needing to ask — treated v2's production code as the source of truth for endpoint contract, per AGENTS.md's "Source of Truth: production code > project notes > general knowledge."
- v3 already has this exact two-phase `verify:true`+`codes` pattern precedented in `src/components/custom/login/login-form.tsx` (2FA login) — de-risked the approach further.
- All i18n keys needed (`security.verification`, `verificationDesc`, `retrySendCode`, `expireDate`, `emailVerificationCode`, etc., plus `actions.confirm`/`close`/`verificationCode*`) already exist in `src/i18n/languages-data/*.json` — v2's locale strings were already carried over wholesale. No translation work needed.
- v3's shadcn `input-otp.tsx` differs from v2's flatter API — v3 requires wrapping slots in `InputOTPGroup` (v2's `InputOTPSlot` took `error`/`placeholder` directly with no group wrapper). Modal built against v3's actual primitive, not v2's.

**Changes:**
- `src/features/documents/api/client.ts` — added `approveContract`/`reviewContract` + `ApproveContractPayload`/`ReviewContractPayload`/`ContractActionVerifyResult` types.
- `src/features/documents/api/queries.ts` — added `useApproveContract`/`useReviewContract`; invalidate detail query only when the response lacks `methods` (i.e., truly finalized, not a dry-run).
- **New** `src/features/documents/components/detail/SecurityVerificationModal.tsx` — dynamic per-method OTP form (react-hook-form + Controller, supports multiple simultaneous methods like v2), countdown timer (`date-fns` `differenceInSeconds`), resend-on-expiry.
- `src/features/documents/components/ContractDetail.tsx` — owns the verify-flow state (`verifyRequest`), builds approve/review payloads from its existing `fields` state, wires modal submit/retry to the mutations, toasts success/failure (local `errMsg` helper, matching the codebase's existing copy-pasted-per-file convention — there's no shared export for it anywhere else either).
- `ActionButtons.tsx` — comment-only update reflecting that OTP is now real; `onDigitalSign` remains the only deferred placeholder (2d-3).

**Explicitly out of scope (deferred, not forgotten):**
- `signature-input` (draw/select signature UI) — belongs to 2d-3; 2d-2b already presets `signatureImgUrl` onto SIGNATURE fields.
- Supplement-file upload on `successSend` (v2's participant1-only branch) — not mentioned in this phase's plan, no v3 endpoint ported, YAGNI.

**Verification:** `npx tsc --noEmit` 0 errors · `npm run lint` 0 errors (4 pre-existing warnings in unrelated files) · `npx vitest run` 28/28. **Not verified in-browser** — needs an authenticated session against a real SIGN_PENDING/REVIEW_PENDING contract, which wasn't available this session (same limitation noted in the previous worklog entries for this plan).

**Next:** user explicitly deferred 2d-3 (digital signature) and 2d-4 (payment) — "do them much later, just note it for now." Plan doc updated to reflect this (both marked ⏸️ deferred, not active). No further Phase2d work planned until user resumes it; when resumed, do a manual browser QA pass on 2d-2c's OTP flow first (untested), then continue from 2d-3.
