---
title: "Dashboard Design Guidelines"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/Design Guidelines.md"
---

# Dashboard Design Guidelines

These rules capture the current dashboard design direction after the compact minimal redesign pass.
Use them whenever changing `src/app/dashboard/**`, `src/components/dashboard/**`, `src/components/products/**`, or `src/components/billing/**`.

## Design Intent

The dashboard is an operational product surface for store owners and admins who scan dense product, billing, import, and conversation data repeatedly.
It should feel quiet, compact, and work-focused. Avoid marketing-page spacing, oversized cards, decorative visuals, and large call-to-action treatment inside dashboard routes.

## Core Rules

- Keep the existing shadcn `radix-luma` design system and semantic tokens. Use `bg-background`, `bg-card`, `bg-muted`, `text-muted-foreground`, `border`, `primary`, `destructive`, and shadcn variants instead of raw custom colors.
- Use flat surfaces by default: `shadow-none`, `border`, and `rounded-xl` for Cards. Do not add heavy shadows, gradients, glass effects, decorative blobs, or large colored panels in dashboard UI.
- Prefer compact page rhythm: route wrappers should generally use `gap-3` or `gap-4`, not `gap-6+`, unless a page has a real multi-section workflow that needs separation.
- Keep headings proportional to dashboard density: page titles usually use `text-xl`; metric values usually use `text-xl` or `text-base`, not `text-3xl+`.
- Use `Card size="sm"` for dashboard cards unless there is a clear reason for the default card spacing.
- Keep cards at `rounded-xl` or smaller. Inner panels, list rows, upload zones, and input overrides should generally be `rounded-lg`.
- Tables should be the primary layout for dense lists. Wrap tables with `overflow-hidden rounded-lg border bg-background` or Card content with `overflow-hidden rounded-lg border p-0`.
- Avoid cards inside cards when possible. If a secondary grouping is needed inside a card, use a flat `rounded-lg border bg-muted/20 p-3` panel.
- Buttons in dashboard workflows should usually be `size="sm"` with `className="h-8"` or `h-9` for primary page actions. Reserve large buttons for public/marketing pages.
- Icons inside shadcn `Button` should use `data-icon="inline-start"` or `data-icon="inline-end"` and should not carry manual `size-*` classes unless outside a shadcn button.
- Forms should be constrained and scannable: use `max-w-lg` or `max-w-2xl`, `gap-3`, compact submit rows, and `rounded-lg` input/textarea overrides where the global component radius feels too pill-shaped for dense settings UI.
- Upload and image controls should not dominate the screen. Prefer compact upload zones (`h-24` style scale) and image thumbnails around `size-24` unless the user needs image inspection.
- Badges should communicate status only. Do not use badges as decoration.
- Empty, loading, and pending states should match final layout density. Skeleton cards should be shorter than content cards when possible.

## Route-Specific Guidance

- Dashboard overview: summary cards should be flat metric tiles with compact copy. Avoid gradients and large numeric display.
- Products: prioritize table scanning, small row actions, compact search, and restrained image thumbnails.
- Import: upload, mapping, progress, and error tables should be dense and procedural. Avoid large instruction cards.
- Conversations: chat detail can keep message bubbles, but use compact radii/padding and avoid oversized avatar/icon treatments.
- Billing: summary state should read like a compact status strip or small flat cards. Payment actions should be clear but not oversized.
- Settings: settings pages should use narrow, flat cards (`max-w-2xl`) and concise forms. Owner-only actions should stay compact.
- Store creation/onboarding: keep the form focused and calm. Do not reuse public landing-page CTA sizing.

## Review Checklist

Before finishing dashboard UI work, check:

- No new raw color palette or non-semantic Tailwind colors for core surfaces.
- No `shadow-md`, `shadow-lg`, `shadow-xl`, gradient cards, or decorative background effects.
- No unnecessary `gap-6+`, `text-3xl+`, `rounded-2xl+`, or large button heights in dashboard routes.
- No icon `size-*` class inside shadcn `Button` children.
- Tables, forms, upload zones, and pending states remain readable on small screens.
- `npm.cmd run lint -- <touched files>` passes before handoff.

## Current Implementation Reference

The current compact direction is reflected in:

- `src/app/dashboard/[storeId]/layout.tsx`
- `src/app/dashboard/[storeId]/page.tsx`
- `src/app/dashboard/[storeId]/billing/page.tsx`
- `src/app/dashboard/[storeId]/products/**`
- `src/app/dashboard/[storeId]/settings/**`
- `src/components/dashboard/**`
- `src/components/products/**`
- `src/components/billing/**`
