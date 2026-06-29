---
title: "Overview"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/Overview.md"
---

# Overview

This note is a short English index. For a detailed Mongolian explanation, read `[[AI-Assistant-for-Onlineshop-Mongolian-Overview]]`.

## Start Here

- `[[AI-Assistant-for-Onlineshop-Mongolian-Overview]]` - detailed project overview in Mongolian
- `[[AI-Assistant-for-Onlineshop-Workflows]]` - key runtime workflows
- `[[AI-Assistant-for-Onlineshop-API-and-Webhooks]]` - API and webhook behavior
- `[[AI-Assistant-for-Onlineshop-Services-Map]]` - service layer map
- `[[AI-Assistant-for-Onlineshop-Deployment-and-Ops]]` - deployment and operations notes

## At a Glance

- The project is an AI sales assistant SaaS for online stores.
- Main flows include Facebook Page connect, Messenger chat, product catalog grounding, human handoff, billing, and observability.
- The `gap-improvement-plan` branch improved chatbot grounding, handoff handling, image attachments, webhook reliability, and Redis pressure.

## Main Areas

- Public pages
- Auth flows
- Store dashboard
- Admin / manage dashboard
- API routes

## Core Modules

- `src/services/chatbot/pipeline.ts`
- `src/services/meta/oauth.ts`
- `src/services/meta/send-api.ts`
- `src/lib/auth/store-access.ts`
- `src/lib/supabase/admin.ts`

## Key Checks

- `requireStoreRole()` is used on protected store paths.
- Redis is kept to short-lived state and cache.
- AI replies are grounded in the product catalog.
- Handoff state works as expected.
- Webhook signature verification is enforced.

