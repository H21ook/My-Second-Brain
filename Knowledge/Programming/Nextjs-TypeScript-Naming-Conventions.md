---
title: "Next.js + TypeScript Naming Convention Standard"
type: knowledge
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - knowledge
  - imported
  - programming
  - nextjs
  - typescript
source_path: "D:/own/obsidian-vaults/Best practices/nextjs-naming-convention-standard.md"
---

# Next.js + TypeScript Naming Convention Standard

## Components → PascalCase
- UserCard.tsx
- ProductTable.tsx
- DashboardHeader.tsx

## Hooks → camelCase + use prefix
- useAuth.ts
- useUser.ts
- useProducts.ts
- useDebounce.ts

Avoid:
- UseAuth.ts
- use-auth.ts

## Services → kebab-case
- auth.service.ts
- product.service.ts
- facebook.service.ts
- gemini.service.ts

## Repositories
- user.repository.ts
- product.repository.ts
- message.repository.ts

## Schemas
- user.schema.ts
- login.schema.ts
- product.schema.ts

## Types
- user.types.ts
- product.types.ts
- auth.types.ts

## Utils / Helpers
- date-format.ts
- currency-format.ts
- slugify.ts

## Constants
- app.constants.ts
- auth.constants.ts

## Next.js App Router Reserved Files
- page.tsx
- layout.tsx
- loading.tsx
- error.tsx
- not-found.tsx
- route.ts
- template.tsx
- default.tsx

## Folder Names
Use kebab-case:
- user-management
- facebook-integration
- product-import

## Recommended Enterprise Standard

| Type | Convention |
|------|------------|
| Components | PascalCase |
| Hooks | camelCase (use*) |
| Services | kebab-case |
| Schemas | kebab-case |
| Types | kebab-case |
| Utils | kebab-case |
| Constants | kebab-case |
| Folders | kebab-case |
| App Router Files | Next.js reserved names |
