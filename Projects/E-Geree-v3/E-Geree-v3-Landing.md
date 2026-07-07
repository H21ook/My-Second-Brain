---
title: "E-Geree-v3 Landing"
type: project
status: draft
created: 2026-07-07
updated: 2026-07-07
tags:
  - project
  - project/e-geree-v3
  - landing
  - reference
source: "5 landing plan (Design-Solution, Redesign-Migration, Phase0-2, Phase3, Phase4-5) distill — 2026-07-07, код баталгаатай"
---

# E-Geree-v3 Landing — Тогтмол лавлагаа

[[E-Geree-v3-Home]] · холбоотой: [[E-Geree-v3-Architecture]] · [[E-Geree-v3-Networking-BFF]] · [[E-Geree-v3-Routing]]

Landing (`/` хуудас) 2026-07-07-нд v2-с restructure + redesign хийгдэж бүрэн шинэчлэгдсэн.
Энэ note нь устгагдсан 5 плангийн **тогтвортой үнэнүүдийг** (бүтэц, дизайн дүрэм, дата урсгал,
бизнес шийдвэр) нэгтгэсэн лавлагаа. Бүх зам/мөр 2026-07-07-ны кодоор баталгаажсан.

---

## 1. Хуудасны бүтэц (секцийн дараалал)

Composition: `src/app/[locale]/(public)/(home)/page.tsx` — server component, дата татаад
секцүүдийг угсарна (page.tsx:32-59). Дараалал = AIDA логик (гол бүтээгдэхүүн дээшээ,
хүнд whitepaper контент доошоо).

| # | Секц | Компонент (`src/components/custom/home/`) | Тэмдэглэл |
|---|------|-------------------------------------------|-----------|
| 1 | Hero | `hero.tsx` | `h1#hero-title`, сүүлийн үг gradient, CTA хос |
| 2 | Trust strip | `trust-strip.tsx` → `partnerships-wrapper.tsx`/`partnerships.tsx` | Партнер marquee (`react-fast-marquee`) |
| 3 | Why E-Geree | `why-egeree.tsx` | v2-ийн 2 advantage блокийг нэг bento болгосон (4 card + 6 icon) |
| 4 | How it works | `how-it-works.tsx` | `easyContract` — CTA → `/contract/create` |
| 5 | Impact stats | `impact-stats.tsx` | Signature секц: `bg-primary` банд + 4 animated counter |
| 6 | Pricing | `pricing-section.tsx` + `pricing-data.ts` | `id="pricing"` (pricing-section.tsx:48), MONTHLY/ANNUAL toggle |
| 7 | FAQ | `faq-section.tsx` | `id="faq"` (faq-section.tsx:65), v2-д байгаагүй ШИНЭ секц + FAQPage JSON-LD |
| 8 | Report band | `report-band.tsx` | SAVE 10X нимгэн CTA (v2 Swiper карусель зориуд хасагдсан) |
| 9 | Certifications | `certifications.tsx` | 4 trust badge (Digital Night, ISO 27001, TSA, GDPR) |
| — | Footer | `footer.tsx` | `(public)/layout.tsx`-д `<main>`-ий дараа, global |

Header: `header.tsx` (nav i18n, anchor: `/#pricing`, `/#faq`, `/blog`, auth-тай үед
`/documents/received`) + `mobile-header.tsx` (Register CTA → `/login`). Секц бүр
`<section aria-labelledby="<id>-title">`, anchor секцүүд `scroll-mt-24`-тэй.

**v2-с ухамсартай хасалтууд (бизнес шийдвэр, буцаахгүй бол хэвээр):** TenXReport Swiper
карусель → нимгэн band; ReportPage; legacy `Partners` компонент; subscription payment
modal (Buy → subscription хуудас руу).

## 2. Design token / branding дүрэм

- **Brand Blue `#004DD9` давамгай.** Hex утга зөвхөн `src/app/globals.css` `@theme`-д
  амьдарна: `--color-brand-blue/purple/teal` (globals.css:57-59). Компонент дотор
  **hex hardcode хориотой** — semantic token л (`bg-primary`, `text-muted-foreground`,
  `bg-card`, `border-border`, `bg-system-{primary,success,warning,purple}/10`).
- **Gradient `#004DD9→#9747FF→#00C7BE`** = `.text-brand-gradient` utility
  (globals.css:285). **Зөвхөн гарчиг/акцентад, яг 2 газар:** hero гарчгийн сүүлийн үг
  (hero.tsx:40) + report band бүтэн гарчиг (report-band.tsx:17). Нэмж хэрэглэхгүй;
  **логон дээр gradient/эффект хатуу хориотой** (v2 brand guideline 2025-02).
- **Фон давхаргууд token-с `color-mix(in oklab, ...)`-ээр гарна** — `.bg-dot-grid`,
  `.bg-brand-glow` (globals.css:297-315) тул dark mode автомат; давхаргууд бүгд
  `pointer-events-none`, статик.
- **Corner-radius lock:** `--radius: 0.625rem` scale; card систем `rounded-2xl` —
  өөрчлөхгүй. Card hover: `hover:-translate-y-1 hover:shadow-elevated-soft`;
  CTA hover: `-translate-y-0.5`.
- **Font:** Manrope (`font-sans`/`font-heading`). Секц контейнер:
  `mx-auto max-w-7xl px-4 md:px-8`, босоо зай `py-16 md:py-24`.
- **Tailwind v4:** `tailwind.config.*` ҮГҮЙ — токен зөвхөн `globals.css` `@theme inline`.
- Секц гарчгийн нэгдсэн хэлбэр: `section-heading.tsx` (eyebrow + h2 + тайлбар).

## 3. Shared primitives

- **`reveal.tsx`** — `<Reveal delay className>`: viewport-д ороход fade + y:24, once.
  `useReducedMotion` үед plain `<div>` буцаана (reveal.tsx:19-21). Бүх секц үүгээр ороогдсон.
- **`animated-counter.tsx`** — `<AnimatedCounter value duration>`: 0→value тоолно
  (`motion/react` `animate`). Reduced-motion үед эцсийн утгыг **render дээр derive**
  хийнэ (animated-counter.tsx:33-37) — effect дотор setState хийвэл
  `react-hooks/set-state-in-effect` lint ERROR болдог нь батлагдсан сургамж.
  `aria-label`-д бүрэн утга.
- **`src/lib/format-count.ts`** — `formatCount()`: en-US мянгатын таслал, max 1 бутархай
  (locale-с үл хамаарах нь дизайны шийдвэр). Colocated vitest тесттэй.
- Анимацийн түвшин: **subtle premium** — scroll-pinning/parallax/scrub үгүй; smooth scroll
  зөвхөн `prefers-reduced-motion: no-preference` дор (globals.css:321-324).

## 4. Дата урсгал (landing service)

**BFF route ХЭРЭГГҮЙ** — хоёр endpoint хоёулаа anonymous GET тул server component-с
`serverFetcher.public`-ээр шууд дуудна (`src/lib/services/landing.ts:9-21`;
base = `getBackendUrl()`, `src/core/config/index.ts:16`):

- `getTotalEnvironmentStat()` → `GET {BACKEND_URL}/environmental-statistic`
- `getSubscriptionPlanList()` → `GET {BACKEND_URL}/subscription/plan-list`

Types: `src/types/landing-types.ts` (`EnvironmentStat`, `SubscriptionPlan`,
`SubscriptionPlanType = CITIZEN | START_UP | ORGANIZATION | CORPORATE | CONTACT_US`).

**Тогтсон бизнес дүрмүүд:**
- Нэгж хөрвүүлэлт (v2-тэй ижил): `savedCarbonSize/1000`, `savedTrashSize/1000` → тонн;
  tree/water шууд (impact-stats.tsx).
- `CONTACT_US` план: API-с `monthPrice`/`amountMap` **null** ирдэг → nullable type,
  `calculatePlanPrice()` null буцаана → үнэний оронд "холбогдох" CTA (`tel:`).
- Үнийн формула (`src/lib/pricing-calc.ts`, v2 `calculateDiscount` порт): MONTHLY =
  `monthPrice` − discount(month=1)%; ANNUAL = `amountMap["12"]` − discount(month=12)%.
- **START_UP tier landing дээр нуугдана** (pricing-section.tsx:19 `HIDDEN_TYPES`).
- Fetch fail үед graceful: stat → бүх утга 0 (`EMPTY_STAT`), plans хоосон бол Pricing
  секц огт render хийхгүй (page.tsx:45).
- Pricing фичер матриц: `pricing-data.ts` (`PRICING_FEATURES`, v2 `static-data.js`-ийн
  1:1 порт; TEXT утга нь орчуулгын key эсвэл `"upTo:N"` кодлол).

## 5. LANDING_LINKS constants

Гадаад линкүүдийн ганц эх сурвалж — `src/lib/constants.ts:13-17`:

```ts
export const LANDING_LINKS = {
  appDownload: "https://onelink.to/2jsk47",
  reportPdf: "https://cdn.e-geree.mn/static/report.pdf",
  salesPhone: "tel:+97699071858",
} as const;
```

CTA route дүрэм: hero `buttonBlue` → `/login`; `buttonGrey` → `LANDING_LINKS.appDownload`;
Pricing Buy → `/profile/user/subscription` (нэвтрээгүйг middleware login руу чиглүүлнэ);
CONTACT_US → `LANDING_LINKS.salesPhone`.

## 6. i18n

- **4 locale:** `src/i18n/languages-data/{mn,en,kr,cn}.json` — landing-ийн бүх key
  4 locale-д sync байх ёстой (mn = эх сурвалж).
- Landing namespace-ууд: `homePage`, `header`, `easyContract`, `save10x`, `statistics`,
  `tree`, `reward`, `subscription`, `pricing`, `staticData.features`, `footer`, `faq`,
  `metadata`.
- **Guard тест:** `src/i18n/languages-data/landing-i18n.test.ts` — REQUIRED key map +
  гүн бүтэц (features/faq категори/`advantageArrays`) 4 locale бүрд assert хийнэ;
  key уствал CI унана. Шинэ landing key нэмбэл энэ тестэд заавал бүртгэнэ.
- Хэв маяг: server секц `await getTranslations(...)`, client секц `useTranslations(...)`;
  server-с client рүү орчуулгыг props-оор дамжуулж болно (`mobile-header.tsx`-ийн
  `registerLabel` pattern).

## 7. SEO

- **Sitemap** (`src/app/sitemap.ts:7`): base = `NEXT_PUBLIC_SITE_URL || "https://e-geree.mn"`
  — robots/metadata-тай **ижил env var** (өмнө нь `NEXT_PUBLIC_BASE_URL` хэрэглэж dev-д
  localhost гарч байсныг нэгтгэсэн). Routes = бодит route-ууд л: `"", "/login", "/blog"`
  × 4 locale.
- **Site JSON-LD:** `src/components/custom/home/site-jsonld.tsx` — нэг `@graph`-д
  `Organization` + `WebSite` + `SoftwareApplication`; root layout-д залгагдсан
  (`src/app/[locale]/layout.tsx:117`). `FAQPage` JSON-LD нь `faq-section.tsx` дотор
  тусдаа — давхардахгүй.
- **PWA maskable icons:** `public/icon-192.png`, `public/icon-512.png` (лого 60% төвд,
  maskable safe zone; `sharp`-аар generate хийсэн — sharp нь Next-ийн dependency,
  package.json-д нэмээгүй). `src/app/manifest.tsx`-д maskable + any бүртгэлтэй.
- **Favicon сургамж:** Next.js `src/app/favicon.ico`, `apple-touch-icon.png`-г
  **автоматаар илрүүлж** serve хийдэг тул manifest-д `favicon.ico` давхар бүртгэвэл
  Chrome "resource size" warning өгдөг — manifest-с зориуд хассан. Manifest-д зөвхөн
  PNG icons үлдээнэ.
- hreflang: metadata API-ийн alternates (ko/zh зөв кодтой); next-intl-ийн автомат
  alternate `Link` header унтраасан — metadata hreflang л үлддэг.

## 8. QA baseline (2026-07-07 хэмжилт)

- **Lighthouse (prod build, desktop): A11y 100 · Best Practices 100 · SEO 100**;
  Perf: LCP 1278ms, CLS 0.00, TTFB 7ms — Core Web Vitals бүгд ногоон.
  Regression гарвал энэ baseline-с доошлохгүй байх зорилт.
- 4 locale render ✓ (CJK pricing card overflow 0), dark mode token-оор автомат,
  reduced-motion бүх анимацид guard-тай (Reveal / AnimatedCounter / smooth-scroll).
- Тест ажиллуулах: `npm test` · `npm run lint` · `npx tsc --noEmit` · `npm run build`.

---

Холбоотой plan: [[E-Geree-v3-Landing-Phase6-Visual-Polish-Plan]] — дараагийн шатны
visual polish plan (секцийн фон давхарга, bento diversity, hero ambient); түүний
`.bg-dot-grid`/`.bg-brand-glow` utility-ууд globals.css:297-315 болон hero/how-it-works/
impact-stats/report-band дээр кодод орсон нь баталгаажсан.
