---
title: "Nextjs-Folder-Structure"
type: knowledge
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - knowledge
  - imported
  - programming
  - nextjs
source_path: "D:/own/obsidian-vaults/Best practices/nextjs-folder-structure.md"
---

Next js project best practices example file structure
## Folder structure

```src/
├─ app/                         # Routing only
│  ├─ (public)/
│  │  └─ page.tsx
│  ├─ (auth)/
│  │  ├─ login/page.tsx
│  │  └─ register/page.tsx
│  ├─ (protected)/
│  │  ├─ dashboard/page.tsx
│  │  └─ layout.tsx
│  ├─ api/
│  │  └─ health/route.ts
│  ├─ layout.tsx
│  └─ globals.css
│
├─ features/                    # Business features
│  ├─ auth/
│  │  ├─ components/
│  │  ├─ hooks/
│  │  ├─ services/
│  │  ├─ schemas/
│  │  ├─ types/
│  │  └─ index.ts
│  │
│  ├─ contracts/
│  │  ├─ components/
│  │  ├─ hooks/
│  │  ├─ services/
│  │  ├─ schemas/
│  │  ├─ types/
│  │  └─ index.ts
│  │
│  └─ documents/
│     ├─ components/
│     ├─ hooks/
│     ├─ services/
│     ├─ schemas/
│     ├─ types/
│     └─ index.ts
│
├─ shared/                      # Reusable, domain-free code
│  ├─ ui/                       # Button, Dialog, Table, FormField
│  ├─ hooks/                    # useDebounce, useMediaQuery
│  ├─ lib/                      # cn, dateFormat, currencyFormat
│  ├─ constants/
│  └─ types/
│
├─ core/                        # App foundation
│  ├─ config/                   # env, app config
│  ├─ http/                     # fetcher, API client
│  ├─ auth/                     # session helpers
│  ├─ errors/                   # AppError, error mapper
│  └─ logger/
│
├─ store/                       # global state only if needed
├─ middleware.ts
└─ instrumentation.ts
```

## Import direction diagram

```
features/contracts/
├─ components/
│  ├─ ContractCard.tsx
│  └─ ContractForm.tsx
├─ hooks/
│  └─ useContracts.ts
├─ services/
│  └─ contract.service.ts
├─ schemas/
│  └─ contract.schema.ts
├─ types/
│  └─ contract.types.ts
└─ index.ts
```