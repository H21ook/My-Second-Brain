---
title: "Next.js + TypeScript Кодын Стандарт ба Rules"
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
source_path: "D:/own/obsidian-vaults/Best practices/nextjs-coding-standards.md"
---

# Next.js + TypeScript Кодын Стандарт ба Rules

> Зорилго: Next.js App Router, TypeScript, React, shadcn/ui, Supabase зэрэг stack ашиглаж буй төсөлд кодын чанар, бүтэц, import direction, security, maintainability-ийг тогтвортой барих стандарт.

---

## 1. Үндсэн зарчим

### 1.1 Кодын чанарын зорилго

- Код уншихад ойлгомжтой, нэг хэв маягтай байна.
- TypeScript type safety-г аль болох бүрэн ашиглана.
- `any`, implicit type, magic string, duplicate logic-оос зайлсхийх.
- Feature-based structure баримтална.
- App Router-ийн `app/` folder-ийг routing-only байлгана.
- Business logic-ийг page/component дотор биш `features/*/services`, `core/*`, `shared/*` дотор байрлуулна.

### 1.2 Заавал мөрдөх автомат шалгалтууд

Project дээр дараах command-ууд алдаагүй ажиллах ёстой.

```bash
pnpm lint
pnpm typecheck
pnpm format:check
pnpm test
pnpm build
```

`package.json` scripts санал:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "test": "vitest run",
    "test:watch": "vitest"
  }
}
```

---

## 2. Folder structure стандарт

Төслийн үндсэн бүтэц:

```txt
src/
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
│  └─ [feature name]/
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
├─ store/                       # Global state only if needed
├─ middleware.ts
└─ instrumentation.ts
```

### 2.1 `app/` folder-ийн дүрэм

`app/` дотор зөвшөөрөх зүйлс:

- `page.tsx`
- `layout.tsx`
- `loading.tsx`
- `error.tsx`
- `not-found.tsx`
- `route.ts`
- Route group folders: `(public)`, `(auth)`, `(protected)`

`app/` дотор хийхгүй зүйлс:

- Том business logic
- Complex form logic
- API client implementation
- Domain service
- Reusable UI component collection

Жишээ:

```tsx
// src/app/(protected)/dashboard/page.tsx
import { DashboardPage } from '@/features/dashboard';

export default function Page() {
  return <DashboardPage />;
}
```

---

## 3. Import direction дүрэм

### 3.1 Зөв import чиглэл

```txt
app -> features -> core/shared
features -> core/shared
core -> shared
shared -> no business dependency
```

### 3.2 Хориглох import

```txt
shared -> features        ❌
core -> features          ❌
features/auth -> features/orders дотор шууд гүн import хийх ❌
app дотор feature service logic бичих ❌
```

Feature хооронд заавал public API буюу `index.ts`-ээр import хийнэ.

```ts
// Зөв
import { LoginForm } from '@/features/auth';

// Буруу
import { LoginForm } from '@/features/auth/components/login-form';
```

---

## 4. TypeScript стандарт

### 4.1 `tsconfig.json` санал

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "es2022"],
    "allowJs": false,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    },
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 4.2 TypeScript дүрэм

- `any` ашиглахгүй. Үнэхээр шаардлагатай бол `unknown` ашиглаад type narrowing хийнэ.
- API response, form data, env, route params-д type тодорхой заана.
- Interface/type naming ойлгомжтой байна.
- Nullable data-г ил тод model-лоно.

```ts
// Зөв
type UserRole = 'superadmin' | 'store_owner' | 'store_admin';

interface StoreUser {
  id: string;
  email: string;
  role: UserRole;
  storeId: string | null;
}

// Буруу
const user: any = data;
```

---

## 5. ESLint стандарт

### 5.1 Санал болгох packages

```bash
pnpm add -D eslint prettier eslint-config-prettier eslint-plugin-import eslint-plugin-unused-imports @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

### 5.2 `.eslintrc.json` жишээ

```json
{
  "extends": [
    "next/core-web-vitals",
    "next/typescript",
    "prettier"
  ],
  "plugins": ["unused-imports", "import"],
  "rules": {
    "no-console": ["warn", { "allow": ["warn", "error"] }],
    "no-debugger": "error",
    "prefer-const": "error",
    "eqeqeq": ["error", "always"],
    "curly": ["error", "all"],

    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": "off",
    "@typescript-eslint/consistent-type-imports": [
      "error",
      { "prefer": "type-imports" }
    ],
    "@typescript-eslint/no-floating-promises": "warn",

    "unused-imports/no-unused-imports": "error",
    "unused-imports/no-unused-vars": [
      "warn",
      {
        "vars": "all",
        "varsIgnorePattern": "^_",
        "args": "after-used",
        "argsIgnorePattern": "^_"
      }
    ],

    "import/order": [
      "error",
      {
        "groups": [
          "builtin",
          "external",
          "internal",
          "parent",
          "sibling",
          "index",
          "type"
        ],
        "newlines-between": "always",
        "alphabetize": {
          "order": "asc",
          "caseInsensitive": true
        }
      }
    ]
  }
}
```

---

## 6. Prettier стандарт

### 6.1 `.prettierrc`

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### 6.2 `.prettierignore`

```txt
.next
node_modules
dist
build
coverage
pnpm-lock.yaml
package-lock.json
yarn.lock
```

---

## 7. Naming convention

### 7.1 File naming

```txt
components/button.tsx              ✅ kebab-case
components/user-profile-card.tsx   ✅ kebab-case
services/auth.service.ts           ✅ domain.service.ts
schemas/login.schema.ts            ✅ domain.schema.ts
hooks/use-auth.ts                  ✅ use-name.ts
```

### 7.2 Code naming

```txt
React component        PascalCase     UserProfileCard
Function               camelCase      getUserById
Variable               camelCase      currentUser
Constant               UPPER_CASE     MAX_UPLOAD_SIZE
Type/Interface         PascalCase     UserProfile
Enum-like union type   PascalCase     UserRole
```

---

## 8. React / Next.js Component стандарт

### 8.1 Server component default

Next.js App Router дээр component-ууд default-оор Server Component байна.

Client Component зөвхөн дараах үед ашиглана:

- `useState`, `useEffect`, `useRef` хэрэгтэй
- Browser API ашиглаж байгаа
- Event handler хэрэгтэй
- Client-side library ашиглаж байгаа

```tsx
// Зөв: Server Component
export async function ProductList() {
  const products = await getProducts();

  return <ProductTable products={products} />;
}
```

```tsx
// Зөв: Client Component
'use client';

import { useState } from 'react';

export function ProductSearch() {
  const [query, setQuery] = useState('');

  return <input value={query} onChange={(event) => setQuery(event.target.value)} />;
}
```

### 8.2 Component дүрэм

- Component 150–200 мөрөөс хэтэрвэл задлах.
- UI component business logic агуулахгүй.
- Props type component-ийн ойролцоо байрлана.
- Boolean prop naming: `isOpen`, `hasError`, `canEdit`, `shouldRender`.

```tsx
interface UserCardProps {
  name: string;
  email: string;
  isActive: boolean;
}

export function UserCard({ name, email, isActive }: UserCardProps) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{email}</p>
      {isActive ? <span>Active</span> : <span>Inactive</span>}
    </div>
  );
}
```

---

## 9. Form, Validation, Schema стандарт

### 9.1 Zod ашиглах

Form validation, API input validation, env validation-д Zod ашиглана.

```bash
pnpm add zod react-hook-form @hookform/resolvers
```

```ts
// src/features/auth/schemas/login.schema.ts
import { z } from 'zod';

export const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

export type LoginInput = z.infer<typeof loginSchema>;
```

### 9.2 Form дүрэм

- Client form дээр `react-hook-form` ашиглана.
- Server/API талд заавал schema parse хийнэ.
- Form error message-ийг UI дотор hardcode биш reusable component-оор харуулна.

---

## 10. API Route Handler стандарт

### 10.1 Route handler бүтэц

```ts
// src/app/api/stores/route.ts
import { NextResponse } from 'next/server';

import { createStoreSchema } from '@/features/stores/schemas/create-store.schema';
import { createStore } from '@/features/stores/services/store.service';

export async function POST(request: Request) {
  const body: unknown = await request.json();
  const input = createStoreSchema.parse(body);

  const store = await createStore(input);

  return NextResponse.json({ data: store }, { status: 201 });
}
```

### 10.2 API дүрэм

- Request body-г `unknown` гэж авч schema validation хийнэ.
- API response format нэг стандарттай байна.
- Error-г raw хэлбэрээр client рүү буцаахгүй.
- Secret/env утгыг response-д гаргахгүй.

Response format санал:

```ts
interface ApiSuccess<T> {
  data: T;
  message?: string;
}

interface ApiError {
  error: {
    code: string;
    message: string;
  };
}
```

---

## 11. Error handling стандарт

### 11.1 AppError ашиглах

```ts
export class AppError extends Error {
  constructor(
    public readonly code: string,
    message: string,
    public readonly statusCode = 500,
  ) {
    super(message);
  }
}
```

### 11.2 Error дүрэм

- `throw new Error('...')`-ийг domain/service layer дээр шууд олон газар ашиглахгүй.
- Known error бол `AppError` ашиглана.
- Unknown error-г logger руу бичээд generic message буцаана.
- Client-д stack trace харуулахгүй.

---

## 12. Env стандарт

### 12.1 Env validation

```ts
// src/core/config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NEXT_PUBLIC_SUPABASE_URL: z.string().url(),
  NEXT_PUBLIC_SUPABASE_ANON_KEY: z.string().min(1),
  SUPABASE_SERVICE_ROLE_KEY: z.string().min(1),
});

export const env = envSchema.parse(process.env);
```

### 12.2 Env дүрэм

- Browser дээр хэрэгтэй env зөвхөн `NEXT_PUBLIC_` prefix-тэй байна.
- Secret key-г client component-д import хийхгүй.
- `process.env.X`-ийг project даяар шууд ашиглахгүй, `core/config/env.ts`-ээр дамжуулна.

---

## 13. Supabase стандарт

### 13.1 Client separation

```txt
src/core/supabase/browser.ts      # Client component/browser client
src/core/supabase/server.ts       # Server component/server action client
src/core/supabase/admin.ts        # Service role, server-only
```

### 13.2 Supabase дүрэм

- Service role key зөвхөн server-only module дотор байна.
- Tenant data query бүр `store_id` filter-тэй байна.
- RLS policy-г database талд заавал идэвхжүүлнэ.
- Client-аас ирсэн `storeId`, `userId`, `role`-д шууд итгэхгүй.
- Authenticated user-г server талд session-оос авна.

---

## 14. Security стандарт

### 14.1 Заавал мөрдөх дүрэм

- User input бүрийг validate/sanitize хийнэ.
- Secret key, access token, webhook secret log хийхгүй.
- File upload дээр size, MIME type, extension шалгана.
- API route дээр authentication/authorization шалгана.
- Admin route дээр role check заавал байна.
- Webhook route дээр signature verification хийнэ.

### 14.2 Хориглох зүйлс

```ts
console.log(process.env.SUPABASE_SERVICE_ROLE_KEY); // ❌
console.log(accessToken); // ❌
```

---

## 15. State management стандарт

### 15.1 State сонголт

```txt
Server data      TanStack Query эсвэл Server Component fetch
Form state       React Hook Form
Local UI state   useState/useReducer
Global state     Zustand зөвхөн үнэхээр шаардлагатай үед
URL state        searchParams
```

### 15.2 Дүрэм

- Server data-г global store-д хадгалахгүй.
- Filter, pagination, tab зэрэг shareable state-г URL query-д хадгална.
- Global state ашиглах бол scope жижиг байлгана.

---

## 16. Styling стандарт

### 16.1 Tailwind/shadcn дүрэм

- Reusable primitive UI-г `shared/ui` дотор байрлуулна.
- `cn()` helper ашиглаж class merge хийнэ.
- Long className давтагдвал component болгоно.
- Design token-ийг `tailwind.config.ts` болон CSS variables-аар удирдана.

```ts
// src/shared/lib/cn.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

---

## 17. Testing стандарт

### 17.1 Test төрөл

```txt
Unit test         pure function, mapper, utility
Component test    reusable UI, form behavior
Integration test  service + API logic
E2E test          login, register, core business flow
```

### 17.2 Дүрэм

- Business critical logic test-тэй байна.
- Utility function test-тэй байна.
- Mock хийхдээ external service boundary дээр mock хийнэ.
- Snapshot test-ийг хэтрүүлж ашиглахгүй.

Test file naming:

```txt
user.service.test.ts
login-form.test.tsx
currency-format.test.ts
```

---

## 18. Git / Commit стандарт

### 18.1 Branch naming

```txt
feature/auth-login
feature/store-products
fix/webhook-signature
refactor/dashboard-layout
chore/update-eslint
```

### 18.2 Conventional commits

```txt
feat: add store product import
fix: validate messenger webhook signature
refactor: move auth logic to feature service
chore: update eslint config
docs: add coding standards
```

---

## 19. Code review checklist

Pull request merge хийхээс өмнө:

- [ ] `pnpm lint` pass
- [ ] `pnpm typecheck` pass
- [ ] `pnpm format:check` pass
- [ ] `pnpm build` pass
- [ ] `any` ашиглаагүй
- [ ] Secret key client талд ороогүй
- [ ] API input schema validation-тэй
- [ ] Error handling standard format-тэй
- [ ] Feature import direction зөрчөөгүй
- [ ] Component хэт том болоогүй
- [ ] Reusable logic duplicated болоогүй
- [ ] Critical logic test-тэй

---

## 20. Recommended dependency set

```bash
pnpm add zod clsx tailwind-merge
pnpm add react-hook-form @hookform/resolvers
pnpm add @tanstack/react-query
pnpm add -D prettier eslint-config-prettier eslint-plugin-import eslint-plugin-unused-imports vitest
```

Optional:

```bash
pnpm add zustand
pnpm add @sentry/nextjs
pnpm add pino
```

---

## 21. Minimum required standard

Төсөл дээр хамгийн багадаа дараах стандартуудыг заавал мөрдөнө.

1. TypeScript strict mode асаалттай байх
2. ESLint + Prettier ашиглах
3. `app/` folder routing-only байх
4. Feature-based folder structure ашиглах
5. Import direction дүрэмтэй байх
6. API input бүр Zod schema validation-тэй байх
7. Env validation central file-тэй байх
8. Secret key client талд орохгүй байх
9. Supabase RLS идэвхтэй байх
10. PR бүр lint/typecheck/build pass хийдэг байх

---

## 22. Товч дүгнэлт

Next.js төсөлд ISO шиг ганц албан ёсны “кодын стандарт” гэж байхгүй. Харин дараах зүйлсийг хамтад нь мөрдвөл practical standard болно.

- TypeScript strict rules
- ESLint rules
- Prettier formatting
- Feature-based architecture
- Import boundary rules
- Security rules
- Testing rules
- Git/PR workflow

Эдгээрийг project repository-д `docs/coding-standards.md` хэлбэрээр хадгалаад, багийн бүх гишүүн нэг стандарт болгон ашиглах нь зөв.
