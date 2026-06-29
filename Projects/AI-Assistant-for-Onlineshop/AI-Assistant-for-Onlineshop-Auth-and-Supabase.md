---
title: "Auth and Supabase"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/Auth and Supabase.md"
---

# Auth and Supabase

This note summarizes the authentication and Supabase design based only on:
- `supabase/**`
- `src/lib/supabase/**`
- `src/app/auth/**`
- `src/app/login/**`
- `src/app/register/**`
- `middleware.ts`

> [!note]
> Unknown or uninspected behavior is marked as **Need verification**.

## Auth Flow

1. The public login and signup pages submit to server actions in `src/app/auth/actions.ts`.
2. Email/password sign-in uses `supabase.auth.signInWithPassword()`.
3. Email/password signup uses `supabase.auth.signUp()`.
4. Facebook and Google sign-in use `supabase.auth.signInWithOAuth()` and redirect back through `/auth/callback`.
5. Password reset uses `resetPasswordForEmail()` and also redirects through `/auth/callback` so the PKCE exchange can happen before the update-password page loads.
6. `/auth/callback` exchanges the returned `code` for a Supabase session and then redirects to the requested page, defaulting to `/dashboard`.
7. `/auth/update-password` updates the password from the browser using the client Supabase instance after the reset flow has established a session.
8. The app shell and protected pages then read the session from cookies on the server.

## Session Handling

### Server

- `src/lib/supabase/server.ts` builds a server Supabase client with `createServerClient()`.
- It reads and writes cookies through `next/headers`.
- This is the client used by server components, route handlers, and server actions that need the current user session.

### Browser

- `src/lib/supabase/client.ts` builds a browser Supabase client with `createBrowserClient()`.
- It is used by the password update page, which calls `supabase.auth.updateUser()` directly in the browser.

### Middleware

- `src/lib/supabase/middleware.ts` refreshes the auth session on each request.
- It checks `supabase.auth.getUser()` and blocks unauthenticated access to `/dashboard` and `/manage`.
- If the user is missing, it redirects to `/auth/login`.
- Role checks are not done in middleware; they are handled in layouts and server helpers.

## Role Handling

### App-level guards

- `requireUser()` redirects anonymous users to `/auth/login`.
- `requireStoreRole(storeId, roles)` verifies the user is a store member and checks the requested role set.
- `requireSuperadmin()` checks `profiles.is_superadmin` and redirects to `/dashboard` if false.
- `assertStoreWritable()` blocks write paths when a store is not in a writable subscription state.

### Database roles

- `public.store_role` is `owner` or `admin`.
- `public.is_store_member()`, `public.is_store_owner()`, and `public.is_superadmin()` are `SECURITY DEFINER` helpers used by RLS.
- `public.shares_store_with()` supports profile visibility for co-members.

### Enforcement model

- RLS is the backstop for authenticated users.
- The application also performs stricter checks in server code, especially for owner-only and write-only actions.

## RLS Assumptions

These are the database rules visible in the migrations:

- `profiles`: users can read their own row, co-members can read each other, and superadmin can read all.
- `stores`: store members can read non-deleted stores; owners can update profile-like fields.
- `store_members`: visible to co-members and superadmin; owners manage memberships.
- `facebook_connections`: readable by store members; sensitive token columns are column-revoked.
- `products` and `product_images`: members can read/write; the app enforces extra read-only behavior when subscriptions are not writable.
- `import_jobs` and `import_job_errors`: members can read/create jobs; job processing writes through service role.
- `conversations` and `messages`: members can read; webhook processing writes through service role.
- `ai_usage_logs`, `subscription_events`, and `audit_logs`: readable by members or superadmin as allowed, with service-role writes.
- `payment_requests`: readable by members; writes are intentionally not tenant-facing in the migration comments, even though the app builds owner-facing payment flows.
- `platform_settings`: readable by authenticated users; service-role writes only.
- `service_role` bypasses RLS, but explicit table privileges are still granted.

## Environment Variables Used

### Auth / Supabase

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`
- `SUPABASE_SECRET_KEY`

### App redirect / OAuth

- `NEXT_PUBLIC_APP_URL`

### Cron / webhook / OAuth security

- `CRON_SECRET`
- `META_VERIFY_TOKEN`

### External auth / platform integrations visible around auth flow

- `SUPABASE_AUTH_SMS_TWILIO_AUTH_TOKEN` is referenced in `supabase/config.toml` only as an example env-backed secret.
- `OPENAI_API_KEY` is referenced in `supabase/config.toml` for Supabase Studio AI, not app auth.

## Security Risks

- `createAdminClient()` bypasses RLS completely. Any misuse would be a privilege escalation.
- The middleware only protects `/dashboard` and `/manage`; any new protected route outside those prefixes would need its own guard.
- OAuth redirects depend on `NEXT_PUBLIC_APP_URL` or the request origin. If misconfigured, callbacks can land on the wrong host.
- `/auth/callback` trusts the `code` parameter returned by the provider and assumes the provider session exchange succeeds.
- `/auth/update-password` runs in the browser, so the reset flow depends on a valid Supabase session already being present.
- Password policies are not enforced in app code beyond a minimum length on the client update page; deeper password rules depend on Supabase configuration.
- `facebook_connections.page_access_token` is explicitly protected by column grants, but app code must continue avoiding `select('*')`.
- The seed file contains example credentials and a fake service setup; safe for local dev only.

## Missing Pieces

- Need verification: the contents of `src/app/register/**`. No register route appears to exist in the current tree.
- Need verification: the full implementations of the server actions in `src/app/auth/actions.ts` beyond the behavior visible in the UI pages.
- Need verification: whether any additional middleware exists beyond the `src/lib/supabase/middleware.ts` helper.
- Need verification: the exact Supabase auth provider configuration for Facebook and Google in deployment.
- Need verification: the runtime wiring that actually imports `updateSession()` into the Next.js middleware entrypoint, if any.
- Need verification: whether the app intentionally relies on Supabase email confirmation being disabled in all environments or only in local development.
- Need verification: whether `payment_requests` writes are actually handled via service role or another route/action path not inspected here.
