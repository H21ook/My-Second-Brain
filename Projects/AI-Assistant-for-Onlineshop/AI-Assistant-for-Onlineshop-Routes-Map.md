---
title: "Routes Map"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/Routes Map.md"
---

# Routes Map

This map was built by inspecting only `src/app/**`.

> [!note]
> When a behavior is not directly visible from the route file itself, it is marked as **Need verification**.

## Public Routes

| Route path | Page/API purpose | Server or client component | Related components/libs | Auth requirement if visible | Notes / risks |
|---|---|---|---|---|---|
| `/` | Marketing homepage for the product | Client page (`use client` not present, but this page renders interactive links and marketing sections) | `next/link`, `next/image`, `lucide-react`, `@/components/ui/*`, `@/lib/constants` | None visible | Landing copy appears localized; some text encoding in source suggests possible file encoding issues. |
| `/privacy` | Privacy page | Need verification | Need verification | None visible | Not inspected in this pass. |
| `/not-found` | Custom 404 page | Need verification | Need verification | None visible | Not inspected in this pass. |
| `/error` | Route-level error boundary | Need verification | Need verification | None visible | Error boundary behavior needs verification. |

## Auth Routes

| Route path | Page/API purpose | Server or client component | Related components/libs | Auth requirement if visible | Notes / risks |
|---|---|---|---|---|---|
| `/auth/login` | Email/password and OAuth login entry | Client component | `useActionState`, `sonner`, `@/app/auth/actions`, `@/components/ui/*` | Public | Submits to server actions for email, Facebook, and Google sign-in. |
| `/auth/signup` | User registration entry | Client component | `useActionState`, `sonner`, `@/app/auth/actions`, `@/components/ui/*` | Public | Validates password confirmation on the client before invoking server action. |
| `/auth/forgot-password` | Password reset request form | Client component | `useActionState`, `sonner`, `@/app/auth/actions`, `@/components/ui/*` | Public | Sends reset email; actual delivery flow is in auth actions, not visible here. |
| `/auth/update-password` | Set a new password after reset | Client component | `next/navigation`, `sonner`, `@/lib/supabase/client`, `@/components/ui/card` | Requires a valid Supabase auth session in practice | Uses client Supabase update; if session is missing it surfaces an auth error. |
| `/auth/callback` | OAuth/code exchange callback | Route handler | `next/server`, `@/lib/supabase/server` | Callback from auth provider | Redirect target defaults to `/dashboard`; failure redirects to `/auth/login?error=auth_failed`. |
| `/auth/actions` | Auth server actions for email/OAuth/reset flows | Server actions file | Need verification | Need verification | Not a route page, but used by auth UI. Exact implementations were not inspected. |

## Dashboard Routes

| Route path | Page/API purpose | Server or client component | Related components/libs | Auth requirement if visible | Notes / risks |
|---|---|---|---|---|---|
| `/dashboard` | Resolves current user to newest visible store, or onboarding | Server component | `@/lib/auth`, `@/lib/supabase/server`, `next/navigation` | `requireUser()` | Redirect-only page; no UI of its own. |
| `/dashboard/new` | New store onboarding entry | Need verification | Need verification | Likely authenticated | Only the route file name was visible in this pass; exact behavior needs verification. |
| `/dashboard/[storeId]` | Store dashboard home with summary cards | Server component | `@/lib/auth/store-access`, `@/lib/constants`, `@/lib/supabase/server`, `@/components/manage/usage-chart`, `@/components/ui/card`, `@/components/ui/badge` | `requireStoreRole(storeId)` in layout and page | Shows store status, counts, Facebook connection state, and AI status. |
| `/dashboard/[storeId]/layout` | Store shell, sidebar, subscription banner, profile/store switcher | Server component | `@/components/dashboard/app-sidebar`, `@/components/dashboard/subscription-banner`, `@/components/ui/sidebar`, `@/components/ui/separator`, `@/lib/auth/store-access`, `@/lib/supabase/server`, `@/lib/constants` | `requireStoreRole(storeId)` | Layout-level auth is not sufficient on soft navigation; child pages/actions still guard themselves. |
| `/dashboard/[storeId]/loading` | Loading UI for store segment | Need verification | Need verification | None visible | Not inspected in this pass. |
| `/dashboard/[storeId]/billing` | Billing history and payment request creation | Server component | `@/lib/auth/store-access`, `@/lib/constants`, `@/lib/supabase/server`, `@/components/billing/*`, `@/components/ui/*`, `@/services/qpay/client`, `@/types/domain` | `requireStoreRole(storeId, ["owner"])` | Owner-only. Shows pending payment card or new payment form. |
| `/dashboard/[storeId]/billing/actions` | Billing server actions | Server actions file | Need verification | Likely owner-only via imported guards | Not inspected directly; exact action set needs verification. |
| `/dashboard/[storeId]/conversations` | Conversation list | Server component | `@/lib/auth/store-access`, `@/lib/supabase/server`, `@/components/ui/*` | `requireStoreRole(storeId)` | Paginated list of conversations. |
| `/dashboard/[storeId]/conversations/[conversationId]` | Single conversation view | Need verification | Need verification | Likely authenticated store access | Not inspected directly. |
| `/dashboard/[storeId]/products` | Product catalog list/search/pagination | Server component | `next/image`, `next/link`, `@/lib/auth/store-access`, `@/lib/constants`, `@/lib/supabase/server`, `@/components/products/delete-product-button`, `@/components/ui/*`, `@/types/domain` | `requireStoreRole(storeId)` | Read-only when the subscription is not writable; import/new/delete controls are hidden or gated. |
| `/dashboard/[storeId]/products/new` | New product form | Need verification | Need verification | Likely store-role gated | Not inspected directly. |
| `/dashboard/[storeId]/products/[productId]/edit` | Edit existing product | Need verification | Need verification | Likely store-role gated | Not inspected directly. |
| `/dashboard/[storeId]/products/import` | Excel import workflow entry | Need verification | Need verification | Likely store-role gated | Not inspected directly. |
| `/dashboard/[storeId]/products/import/[jobId]` | Import job detail/progress page | Need verification | Need verification | Likely store-role gated | Not inspected directly. |
| `/dashboard/[storeId]/products/actions` | Product server actions | Server actions file | Need verification | Likely store-role gated | Not inspected directly. |
| `/dashboard/[storeId]/settings` | General settings page for store metadata | Server component | `@/lib/auth/store-access`, `@/components/dashboard/general-settings-form`, `@/types/domain` | `requireStoreRole(storeId, ["owner"])` | Owner-only. Passes writable flag based on subscription status. |
| `/dashboard/[storeId]/settings/members` | Member management UI | Need verification | Need verification | Likely owner/admin gated | Not inspected directly. |
| `/dashboard/[storeId]/settings/ai` | AI settings page | Need verification from this pass, but file naming and imports imply a settings form page | Likely `@/components/dashboard/ai-settings-form` | Likely owner/admin gated | Not directly read in this pass. |
| `/dashboard/[storeId]/settings/facebook` | Facebook connection settings | Need verification | Need verification | Likely owner-only | Not directly read in this pass. |
| `/dashboard/[storeId]/settings/facebook/select-page` | Page selection after Facebook OAuth | Need verification | Need verification | Likely owner-only | Not directly read in this pass. |
| `/dashboard/[storeId]/settings/actions` | Settings server actions | Server actions file | Need verification | Likely owner-only | Not inspected directly. |
| `/dashboard/[storeId]/settings/layout` | Settings sub-layout | Need verification | Need verification | Likely store-role gated | Not inspected directly. |

## Manage Routes

| Route path | Page/API purpose | Server or client component | Related components/libs | Auth requirement if visible | Notes / risks |
|---|---|---|---|---|---|
| `/manage` | Platform admin overview with MRR and usage chart | Server component | `@/lib/auth/store-access`, `@/lib/constants`, `@/lib/supabase/server`, `@/components/manage/usage-chart`, `@/components/ui/card` | `requireSuperadmin()` | Central admin dashboard. |
| `/manage/layout` | Admin shell and nav | Server component | `next/link`, `lucide-react`, `@/lib/auth/store-access`, `@/components/ui/button` | `requireSuperadmin()` | Re-runs guard at layout level; server actions still need their own checks. |
| `/manage/stores` | Store list and activation controls | Server component | `next/link`, `@/lib/auth/store-access`, `@/lib/constants`, `@/lib/supabase/server`, `@/components/manage/activate-store-dialog`, `@/components/ui/*`, `@/types/domain` | `requireSuperadmin()` | Resolves owners via extra query because `store_members.user_id` does not FK-join to `profiles`. |
| `/manage/stores/[storeId]` | Single store admin view | Need verification | Need verification | Likely superadmin-only | Not inspected directly. |
| `/manage/payments` | Payment review queue and bank settings | Server component | `@/lib/auth/store-access`, `@/lib/constants`, `@/lib/supabase/server`, `@/components/manage/bank-settings-form`, `@/components/manage/payment-review-buttons`, `@/components/ui/*` | `requireSuperadmin()` | Shows pending and processed requests. |
| `/manage/audit` | Audit log page | Need verification | Need verification | Likely superadmin-only | Not inspected directly. |
| `/manage/users` | User management page | Need verification | Need verification | Likely superadmin-only | Not inspected directly. |
| `/manage/actions` | Admin server actions | Server actions file | Need verification | Likely superadmin-only | Not inspected directly. |

## API Routes

| Route path | Page/API purpose | Server or client component | Related components/libs | Auth requirement if visible | Notes / risks |
|---|---|---|---|---|---|
| `/api/webhook` | Meta Messenger webhook verification and inbound event intake | Route handler | `next/server`, `@/services/chatbot/pipeline`, `@/services/meta/verify-signature`, `@/services/meta/types` | Signature-based, not user auth | Uses `after()` to process events asynchronously; runtime duration is capped at 60s. |
| `/api/facebook/connect` | Starts Facebook OAuth connection for a store | Route handler | `next/server`, `@/lib/auth/store-access`, `@/services/meta/oauth` | `requireStoreRole(storeId, ["owner"])` | Redirects to Meta OAuth; on error bounces back with `?error=connect`. |
| `/api/facebook/callback` | Facebook OAuth callback and page discovery | Route handler | `next/server`, `@/lib/auth`, `@/services/meta/oauth` | Must match the initiating user session/state | Uses OAuth state as CSRF gate, then stores fetched pages and redirects to page selection. |
| `/api/qpay/callback` | Payment callback hint from QPay | Route handler | `next/server`, `@/lib/payments`, `@/lib/supabase/admin`, `@/services/qpay/client` | No user auth; treated as unauthenticated webhook | Always returns 200 to avoid retries; verifies payment by checking QPay directly. |
| `/api/cron/subscription-lifecycle` | Scheduled lifecycle transition and purge job | Route handler | `next/server`, `@/lib/constants`, `@/lib/supabase/admin` | Bearer token via `CRON_SECRET` | Deletes storage objects and purges deleted stores; `maxDuration = 300`. |
| `/api/stores/[storeId]/import/template` | Download import template | Route handler | Need verification | Likely store-role gated | Not inspected directly. |
| `/api/stores/[storeId]/import/[jobId]/run` | Trigger import job execution | Route handler | Need verification | Likely store-role gated | Not inspected directly. |
| `/api/stores/[storeId]/payments/[paymentId]/check` | Manual payment check endpoint | Route handler | Need verification | Likely store-role gated | Not inspected directly. |

## Folder-Level Notes

- `app/**` is a mixed App Router tree with public pages, authenticated dashboards, admin pages, and route handlers.
- Store-scoped pages consistently use `requireStoreRole()` or rely on the store layout plus page-level checks.
- Admin pages consistently use `requireSuperadmin()`.
- Some route files are clearly client components (`login`, `signup`, `forgot-password`, `update-password`), while most dashboard/admin pages are server components.
- Several route files were not opened in this pass, so their entries are marked `Need verification` rather than guessed.

## Open Questions

- Need verification: exact implementation of `/privacy`, `/not-found`, and `/error`.
- Need verification: exact UI and behavior of `/dashboard/new`, conversation detail, product create/edit/import pages, and admin detail pages.
- Need verification: the server action files under `auth/`, `dashboard/[storeId]/`, and `manage/`.
- Need verification: whether all protected routes rely solely on layout/page guards or also on middleware for runtime enforcement.
- Need verification: whether `/api/stores/[storeId]/payments/[paymentId]/check` and import endpoints require a session or a signed request.
