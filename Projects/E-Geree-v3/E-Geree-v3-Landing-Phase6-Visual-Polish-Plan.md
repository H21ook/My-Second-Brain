---
title: "E-Geree-v3 Landing Phase 6 Visual Polish Plan"
status: done
created: 2026-07-07
updated: 2026-07-07
tags:
  - egeree/plan
  - egeree/landing
---

# E-Geree-v3 Landing — Phase 6 (Visual Polish) Implementation Plan

> [!note] Хэрэгжсэний дараа устгаж болно. Тогтмол reference биш.

[[E-Geree-v3-Home]] · холбоотой: [[E-Geree-v3-Landing]]

> **For agentic workers:** REQUIRED SUB-SKILL: superpowers:subagent-driven-development. Redesign-Preserve горим — брэнд/IA/i18n/copy ХӨНДӨХГҮЙ, зөвхөн визуал давхарга.

**Goal:** design-taste-frontend audit-ын олдворуудыг засах — секцийн фон давхарга (pattern/glow), bento background diversity, hero ambient гүнзгийрүүлэлт, popular pricing ялгарал. Бүгд токен-даяар, dark автомат.

**Design Read:** Redesign-Preserve; trust-first + subtle premium; VARIANCE 6 · MOTION 5 · DENSITY 4. Motion шинээр нэмэхгүй (одоогийн Reveal/counter хангалттай — "motion motivated" дүрэм).

**Tech Stack:** Tailwind v4 (`globals.css` plain class конвенц), `color-mix(in oklab, ...)` (modern browsers OK).

## Global Constraints

- Брэнд: `#004DD9` давамгай хэвээр; gradient зөвхөн гарчиг (одоогийн 2 газраас нэмэхгүй); логон дээр юу ч үгүй.
- Hex hardcode ХОРИОТОЙ — зөвхөн token/`color-mix`/цагаан-alpha overlay (`rgba(255,255,255,...)` highlight зөвшөөрнө — брэнд өнгө биш).
- i18n текст, секцийн бүтэц/дараалал, aria/id, corner-radius систем (rounded-2xl) ӨӨРЧЛӨХГҮЙ.
- Шинэ dependency үгүй; шинэ motion үгүй; pattern давхаргууд бүгд `pointer-events-none`, статик (reduced-motion асуудалгүй).
- Page Theme Lock: dark/light хоёуланд нь ажиллах ёстой (token + color-mix автомат).
- Шалгах: `npm run build` · `npx tsc --noEmit` · `npm run lint` · browser (light+dark) контроллер шалгана.
- Working tree-ийн хамааралгүй өөрчлөлтөд халдахгүй; зөвхөн өөрийн файлаа `git add`.

---

### Task 1: Фон давхаргын utility-ууд (globals.css)

**Files:**
- Modify: `src/app/globals.css` (төгсгөлд, `.text-brand-gradient`-ийн хажууд)

**Interfaces:**
- Produces: `.bg-dot-grid` (нарийн цэгэн тор, foreground-оос color-mix тул dark автомат), `.bg-brand-glow` (дээрээс брэнд хөх зөөлөн туяа). Task 2-4 хэрэглэнэ.

- [ ] **Step 1: Нэмэх**

```css
/* Phase 6: секцийн фон давхаргууд. Токеноос color-mix хийдэг тул dark mode автомат. */
.bg-dot-grid {
  background-image: radial-gradient(
    circle,
    color-mix(in oklab, var(--color-foreground) 8%, transparent) 1px,
    transparent 1px
  );
  background-size: 24px 24px;
}

.bg-brand-glow {
  background-image: radial-gradient(
    60% 50% at 50% 0%,
    color-mix(in oklab, var(--color-brand-blue) 9%, transparent),
    transparent 70%
  );
}
```

- [ ] **Step 2: Шалгах + Commit**

Run: `npm run build` — PASS (CSS parse OK).

```bash
git add src/app/globals.css
git commit -m "feat(theme): bg-dot-grid, bg-brand-glow фон utility (токен-даяар, dark автомат)"
```

---

### Task 2: Hero ambient + trust logo polish

**Files:**
- Modify: `src/components/custom/home/hero.tsx`
- Modify: `src/components/custom/home/partnerships.tsx` (лого img class л)

**Interfaces:**
- Consumes: Task 1 utilities.
- Produces: hero-д брэнд туяа + доош бүдгэрэх цэгэн тор давхарга (Spotlight хэвээр, дээр нь биш доор нь); партнер лого default бүдэг → hover-т тод (амьд social proof мэдрэмж).

- [ ] **Step 1: hero.tsx — section дотор, Spotlight-ийн ӨМНӨ нэмэх**

`<Spotlight />` мөрийн өмнө (section эхний хүүхэд болгож):

```tsx
{/* Ambient фон: брэнд туяа + доош бүдгэрэх цэгэн тор (декоратив, статик) */}
<div aria-hidden className="pointer-events-none absolute inset-0 bg-brand-glow" />
<div
    aria-hidden
    className="pointer-events-none absolute inset-x-0 top-0 h-[480px] bg-dot-grid [mask-image:radial-gradient(70%_100%_at_50%_0%,black_20%,transparent_100%)]"
/>
```

Өөр юу ч өөрчлөхгүй (section аль хэдийн `relative overflow-hidden`).

- [ ] **Step 2: partnerships.tsx — лого img-үүдэд muted → hover тод**

Лого `<Image>`-үүдийн `className`-д (одоогийн `h-12 w-auto object-contain` дээр) нэмэх: `opacity-70 grayscale transition-[opacity,filter] duration-300 hover:opacity-100 hover:grayscale-0`. Light/dark хоёр хувилбарын img хоёуланд нь. Өөр өөрчлөлтгүй.

- [ ] **Step 3: Шалгах + Commit**

Run: `npx tsc --noEmit && npm run lint` — PASS.

```bash
git add src/components/custom/home/hero.tsx src/components/custom/home/partnerships.tsx
git commit -m "feat(home): hero ambient давхарга (glow+dot-grid), партнер лого grayscale-hover"
```

---

### Task 3: Bento background diversity + How-it-works тор

**Files:**
- Modify: `src/components/custom/home/why-egeree.tsx`
- Modify: `src/components/custom/home/how-it-works.tsx`

**Interfaces:**
- Consumes: Task 1 utilities.
- Produces: bento-гийн cell-үүд 3 янзын арьстай (accent+highlight / brand tint / цэвэр card) — design-taste "Bento Background Diversity" дүрэм хангагдана; how-it-works секц цэгэн торон дэвсгэртэй.

- [ ] **Step 1: why-egeree.tsx — том 4 card-ын арьс ялгах**

`cards` render хэсэгт (одоогийн ternary-г өргөжүүлж):

- Card 1 (accent, wide) — одоогийн `bg-primary text-primary-foreground` дээр НЭМЖ дотор нь цагаан highlight: card div-д `relative overflow-hidden` нэмээд, эхний хүүхэд болгож:

```tsx
{card.accent && (
    <div
        aria-hidden
        className="pointer-events-none absolute inset-0 bg-[radial-gradient(120%_100%_at_100%_0%,rgba(255,255,255,0.14),transparent_55%)]"
    />
)}
```

(контентыг `relative` span-д ороох шаардлагагүй — highlight хамгийн доор тул text уншигдана; харагдац муу бол контентоо `relative`-д ороо.)

- Card 4 (wide, non-accent) — `border border-border bg-card` → `border border-primary/20 bg-primary/5` болгож брэнд tint өгнө. Card 2-3 хэвээр (цэвэр card). Ингэснээр 4 card = 3 арьс: accent, tint, цэвэр×2.

Хэрэгжилт: `cards` массивд `tint: boolean` талбар нэмж (`card4: tint: true`), class ternary-д `card.tint ? "border border-primary/20 bg-primary/5 " : ...` салаа нэм.

- [ ] **Step 2: why-egeree.tsx — icon tile 2-т tint**

6 icon tile-ийн 1 ба 5 дахь (Flash, TrailSign)-д `bg-card` → `bg-primary/5`: ICON_FEATURES-т `tint?: true` нэмж (Flash, TrailSign дээр), class-д ternary. Бусад 4 хэвээр.

- [ ] **Step 3: how-it-works.tsx — секцэд цэгэн тор**

Section (одоо `py-16 md:py-24 bg-muted/40`) → `relative` нэмээд эхний хүүхэд:

```tsx
<div
    aria-hidden
    className="pointer-events-none absolute inset-0 bg-dot-grid opacity-60 [mask-image:linear-gradient(to_bottom,transparent,black_15%,black_85%,transparent)]"
/>
```

Контент wrapper div-д `relative` нэм (тор дээгүүр гарахгүй).

- [ ] **Step 4: Шалгах + Commit**

Run: `npx tsc --noEmit && npm run lint` — PASS.

```bash
git add src/components/custom/home/why-egeree.tsx src/components/custom/home/how-it-works.tsx
git commit -m "feat(home): bento cell-үүдэд арьсны ялгаа (accent highlight, brand tint), how-it-works цэгэн тор"
```

---

### Task 4: Impact texture + Pricing popular + Report glow

**Files:**
- Modify: `src/components/custom/home/impact-stats.tsx`
- Modify: `src/components/custom/home/pricing-section.tsx`
- Modify: `src/components/custom/home/report-band.tsx`

**Interfaces:**
- Produces: хавтгай хөх банд гүнтэй болно; popular card статик ялгаралтай; report card брэнд туяатай.

- [ ] **Step 1: impact-stats.tsx — бандад гүн**

Section-д `relative overflow-hidden` нэмээд эхний хүүхэд:

```tsx
{/* Хавтгай брэнд банданд гүн: дээд цагаан туяа + бүдэг цэгэн тор */}
<div
    aria-hidden
    className="pointer-events-none absolute inset-0 bg-[radial-gradient(80%_70%_at_50%_0%,rgba(255,255,255,0.10),transparent_60%)]"
/>
<div
    aria-hidden
    className="pointer-events-none absolute inset-0 bg-dot-grid opacity-20"
/>
```

Контент wrapper `mx-auto max-w-7xl...` div-д `relative` нэм. Тэмдэглэл: `.bg-dot-grid` foreground-оос color-mix хийдэг — хөх банд дотор foreground нь хуудасны token хэвээр тул цэгүүд бүдэг саарал болно; hex оруулахгүй, opacity-20 хангалттай.

- [ ] **Step 2: pricing-section.tsx — popular card ялгарал**

`isPopular` card-ын class: одоогийн `border-primary` дээр НЭМЖ `bg-primary/5 shadow-elevated-soft` (статик — hover-ийн өмнө ч ялгарна):

```tsx
(isPopular ? "border-primary bg-primary/5 shadow-elevated-soft" : "border-border")
```

- [ ] **Step 3: report-band.tsx — card-д брэнд туяа**

Дотор card div (`rounded-3xl border border-border bg-card ...`)-д `relative overflow-hidden` нэмээд эхний хүүхэд:

```tsx
<div aria-hidden className="pointer-events-none absolute inset-0 bg-brand-glow" />
```

Гарчиг/товч аль хэдийн стакид дээр нь тул relative нэмэх шаардлага гарвал контент элементүүдэд `relative` нэм.

- [ ] **Step 4: Шалгах + Commit**

Run: `npx tsc --noEmit && npm run lint` — PASS.

```bash
git add src/components/custom/home/impact-stats.tsx src/components/custom/home/pricing-section.tsx src/components/custom/home/report-band.tsx
git commit -m "feat(home): impact банданд гүн, popular card статик ялгарал, report card брэнд туяа"
```

---

### Task 5: Контроллер — визуал верификац + gates + vault

- [ ] Browser: /mn light + dark, mobile viewport — секц бүр screenshot, pattern-ууд subtle эсэх (давамгайлбал opacity бууруул), текст contrast эвдрээгүй эсэх. Pre-Flight: theme lock, accent lock (#004DD9 л), radius lock (rounded-2xl), gradient text 2 газраас нэмэгдээгүй.
- [ ] `npm test && npm run lint && npx tsc --noEmit && npm run build` — PASS.
- [ ] Vault: энэ план status done + worklog; Design-Solution-д Фаз 6 мөр нэмэх шаардлагагүй (энэ план өөрөө бүртгэл).

## Documentation Impact

- [ ] [[E-Geree-v3-Landing]] — design token/section өөрчлөлт орвол шинэчлэх
- [x] Worklog-д гүйцэтгэлийг тэмдэглэх
- [ ] Хэрэгжсэний дараа энэ plan-ыг устгах

## Worklog (2026-07-07)

Бүх 5 task дууссан, subagent-driven (branch `dev-khishigee`, commits `e9a049f..ee36704`):

- **Task 1** `e9a049f` — `.bg-dot-grid`, `.bg-brand-glow` utility (token/color-mix, dark автомат). Review clean.
- **Task 2** `9d741d0` — hero ambient (glow + masked dot-grid), партнер лого grayscale→hover. Review clean.
- **Task 3** `e457582` — bento 3 арьс (accent highlight / brand tint / цэвэр), icon tile 1,5 tint, how-it-works цэгэн тор. Review clean. Тэмдэглэл: `tint: false`-ыг бүх ICON_FEATURES-т ил бичсэн (tsc tuple-union шаардлага); `relative overflow-hidden` 4 card-д нийтэд нь (verified harmless).
- **Task 4** `ee36704` — impact банд гүн (цагаан туяа + dot-grid opacity-20), popular pricing `bg-primary/5 shadow-elevated-soft`, report card `bg-brand-glow` (+content `relative` stacking fix). Review clean.
- **Task 5** — Gates: tests 475/475, lint 0 error, tsc clean, build PASS. Browser: light+dark+mobile(390) секц бүр шалгасан — pattern-ууд subtle, contrast эвдрэлгүй, horizontal overflow үгүй. Pre-Flight: text-gradient 2 газар хэвээр, hex шинээр 0, accent/radius/theme lock хэвээр.
- **Final whole-branch review (e3d7e0c..ee36704): Ready to merge.** Critical/Important үгүй. Тэмдэглэсэн minor-ууд (засах шаардлагагүй гэж шийдсэн): (1) dark-д `bg-primary/5` нь `bg-card`-ыг орлодог тул tint card-ууд бага зэрэг "доошилсон" харагдана — border/badge/shadow ялгарал хангалттай, browser-оор баталсан; (2) impact дээрх dot-grid бараг үл мэдэгдэм (~1.6% alpha) — план-ийн заасан утга; (3) color-mix-гүй хуучин browser fallback ширүүн — төслийн modern-browser floor-т хамаагүй.
