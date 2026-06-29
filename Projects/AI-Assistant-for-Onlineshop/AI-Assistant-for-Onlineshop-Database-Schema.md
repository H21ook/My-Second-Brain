---
title: "Database Schema"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/Database Schema.md"
---

# Database Schema

This note summarizes the database design from `supabase/**`.

## What The Schema Contains

### Core identity and membership

- `profiles`
- `stores`
- `store_members`

### Sales and product data

- `products`
- `product_images`
- `import_jobs`
- `import_job_errors`

### Messaging and AI usage

- `conversations`
- `messages`
- `ai_usage_logs`

### Billing and platform operations

- `payment_requests`
- `platform_settings`
- `subscription_events`
- `audit_logs`
- `facebook_connections`

## Important Types

- `subscription_status`: `inactive`, `active`, `expired`, `pending_deletion`, `deleted`
- `store_role`: `owner`, `admin`
- `import_status`: `uploaded`, `mapped`, `processing`, `completed`, `completed_with_errors`, `failed`
- `message_role`: `user`, `assistant`
- `payment_method`: `manual_transfer`, `qpay`
- `payment_status`: `pending`, `paid`, `rejected`, `canceled`

## Major Behaviors

### Profiles

- One row per auth user.
- Created automatically by trigger on `auth.users` insert.
- Email is synced from auth updates.

### Stores and membership

- `create_store()` is the intended creation path.
- No direct tenant insert policy on `stores`.
- Owners are inserted into `store_members` by the same RPC.
- `stores` has lifecycle columns for deletion and purge tracking.

### Products and imports

- `products.search_text` is a generated column for search.
- SKU uniqueness is scoped per store.
- Import jobs track mapped/uploaded/processing/completed status.
- `import_job_errors` stores row-level validation results.

### Conversations and usage

- Conversations are keyed by `page_id` + `psid`.
- Messages are display-only and mirror the chat turn history.
- AI usage logs are retained for analytics even after store purge.

### Billing

- `payment_requests` stores manual-transfer and QPay billing flows.
- `platform_settings` holds shared platform values like the bank account.

## RLS Model

### Helper functions

- `is_superadmin()`
- `is_store_member(store_id)`
- `is_store_owner(store_id)`
- `shares_store_with(user_id)`

### Table access shape

- `profiles`: self, co-members, superadmin
- `stores`: members can read non-deleted stores; owners can update some fields
- `store_members`: co-members and superadmin can read; owners manage
- `facebook_connections`: members can read status, token column is restricted by grants
- `products` / `product_images`: member read/write with app-layer writable gating on top
- `import_jobs` / `import_job_errors`: members can read/create, processing via service role
- `conversations` / `messages`: member read, webhook writes via service role
- `ai_usage_logs` / `subscription_events`: member read, service role writes
- `audit_logs`: superadmin only
- `payment_requests`: authenticated select; operational writes are handled outside tenant-facing flows
- `platform_settings`: authenticated select; service-role writes

## Lifecycle And Purge

- `run_subscription_lifecycle()` transitions stores through subscription states.
- `purge_store_data()` removes tenant data after deletion while keeping tombstone data for analytics.
- Storage objects are removed by the cron route, not by SQL.

## Storage Buckets

- `product-images` is public and stores store-scoped image files.
- `import-files` is private and stores uploaded import workbooks.

## Missing Pieces

- Need verification: exact frontend use of all tables, especially newer ones such as `payment_requests` and `platform_settings`.
- Need verification: whether any migrations after the visible set changed policies or added columns.
- Need verification: the final production seed / bootstrap process outside local development.
