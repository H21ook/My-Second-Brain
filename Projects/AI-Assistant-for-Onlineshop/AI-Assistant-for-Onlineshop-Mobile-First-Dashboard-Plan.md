---
title: "Mobile First Dashboard Plan"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/ai-assistant-for-onlineshop
source_path: "D:/own/obsidian-vaults/ai-assistant-for-onlineshop/plans/Mobile First Dashboard Plan.md"
---

# Mobile First Dashboard Plan

## Goal

Онлайн дэлгүүрийн эзэд ихэнх ажлаа гар утсаар хийдэг гэж үзнэ.

Тиймээс dashboard-ийн бүх үндсэн үйлдэл:

- бараа нэмэх
    
- бараа засах
    
- үлдэгдэл өөрчлөх
    
- үнэ өөрчлөх
    
- зураг оруулах
    
- чат харах
    
- AI тохиргоо хийх
    
- төлбөр шалгах
    
- Facebook page холбох
    
- тайлан харах
    

гэсэн бүх урсгал mobile дээр бүрэн ажилладаг байх ёстой.

---

# Core Principle

Desktop dashboard-ийг mobile дээр шахаж харуулах биш.

Mobile хэрэглээнд зориулж UX-г дахин зохион байгуулна.

---

# Current App Context

Одоогийн систем нь Next.js App Router dashboard бүтэцтэй бөгөөд store-scoped routes нь `/dashboard/[storeId]/**` доор байрладаг. Product list, product create/edit/import, conversations, billing, settings зэрэг үндсэн хэсгүүд аль хэдийн route түвшинд байгаа.

Database талд `products`, `product_images`, `conversations`, `messages`, `payment_requests`, `facebook_connections` зэрэг mobile dashboard-д хэрэгтэй үндсэн tables байгаа.

---

# Mobile Design Rule

Mobile дээр хэрэглэгч:

> “Компьютергүй байсан ч дэлгүүрээ бүрэн удирдаж чадна”

гэсэн мэдрэмж авах ёстой.

---

# Navigation

## Desktop

Desktop дээр одоогийн sidebar хэвээр үлдэнэ.

## Mobile

Mobile дээр sidebar ашиглахгүй.

Доод tab navigation ашиглана.

Main tabs:

- Нүүр
    
- Бараа
    
- Чат
    
- AI
    
- Илүү
    

---

# Mobile Bottom Tabs

## Нүүр

- Өнөөдрийн чат
    
- AI ашиглалт
    
- Барааны тоо
    
- Facebook connection status
    
- Subscription status
    
- Түргэн үйлдлүүд
    

## Бараа

- Product list
    
- Search
    
- Filter
    
- Add product
    
- Quick stock edit
    
- Quick price edit
    

## Чат

- Conversation list
    
- Customer message detail
    
- AI reply history
    
- Human handoff indicator
    

## AI

- AI status
    
- AI tone
    
- FAQ
    
- Business instructions
    
- Catalog sync status
    

## Илүү

- Settings
    
- Billing
    
- Members
    
- Facebook connection
    
- Import
    
- Reports
    
- Logout
    

---

# Product Mobile UX

## Product List

Desktop table-г mobile дээр шууд харуулахгүй.

Mobile дээр product card list ашиглана.

Card бүр:

- зураг
    
- нэр
    
- үнэ
    
- үлдэгдэл
    
- status
    
- quick actions
    

харуулна.

Quick actions:

- Засах
    
- Үлдэгдэл өөрчлөх
    
- Үнэ өөрчлөх
    
- Зураг солих
    
- Устгах
    

---

# Add Product Mobile UX

Бараа нэмэх хуудас mobile дээр хамгийн түрүүнд сайжрах ёстой.

## Default visible fields

Зөвхөн:

- Зураг
    
- Барааны нэр
    
- Үнэ
    
- Үлдэгдэл
    
- Хадгалах
    

харагдана.

## Secondary fields

“Нэмэлт мэдээлэл” нээхэд:

- Ангилал
    
- SKU
    
- Брэнд
    
- Barcode
    
- Тайлбар
    
- Өртөг үнэ
    

гарна.

## Advanced fields

“Advanced” хэсэгт:

- AI keywords
    
- FAQ
    
- Supplier
    
- Warehouse
    
- Variant
    

байна.

---

# Mobile Image Upload

Mobile дээр зураг оруулах нь маш хурдан байх ёстой.

Support хийх:

- Camera
    
- Gallery
    
- Multiple image upload
    
- Image preview
    
- Reorder images
    
- Remove image
    

Зураг оруулсны дараа хэрэглэгч page солихгүйгээр шууд хадгална.

---

# Quick Edit Pattern

Mobile дээр жижиг өөрчлөлт хийхийн тулд edit page руу заавал оруулахгүй.

Product card дээрээс:

- price
    
- stock
    
- status
    

зэрэгийг bottom sheet ашиглан хурдан засна.

Жишээ:

Product card → “Үлдэгдэл” → bottom sheet → шинэ тоо → хадгалах.

---

# Bottom Sheet Standard

Mobile дээр modal их ашиглахгүй.

Bottom sheet ашиглана.

Ашиглах газрууд:

- Category сонгох
    
- Stock update
    
- Price update
    
- Delete confirm
    
- Image options
    
- Filter
    
- Sort
    
- Bulk actions
    

---

# Search and Filter

Mobile product list дээр:

- sticky search bar
    
- category filter
    
- stock status filter
    
- price sort
    
- newest sort
    

байна.

Filter screen нь full page биш, bottom sheet байна.

---

# Conversations Mobile UX

Conversation list mobile дээр Messenger шиг харагдана.

Conversation card:

- customer name / psid
    
- latest message
    
- unread indicator
    
- AI/Human status
    
- time
    

Conversation detail:

- chat bubbles
    
- AI reply indicator
    
- product mentioned
    
- quick human reply
    
- copy message
    
- mark handled
    

---

# AI Settings Mobile UX

AI тохиргоо mobile дээр хүнд form шиг биш байх ёстой.

Card-based тохиргоо ашиглана.

Sections:

- AI асаах / унтраах
    
- Хариулах tone
    
- Дэлгүүрийн тухай
    
- Хүргэлтийн мэдээлэл
    
- Төлбөрийн мэдээлэл
    
- FAQ
    
- AI fallback message
    

---

# Reports Mobile UX

Mobile report нь table биш summary card байна.

Cards:

- Өнөөдрийн чат
    
- AI хариулсан тоо
    
- Хамгийн их асуусан бараа
    
- Бага үлдэгдэлтэй бараа
    
- Идэвхгүй бараа
    
- Борлуулалтын summary
    

Chart байж болно, гэхдээ эхэндээ card-first байна.

---

# Mobile Import Strategy

Excel import mobile дээр заавал гол flow биш.

Гэхдээ дараах боломж байна:

- Import хуудас mobile дээр уншигдахуйц байх
    
- Template татах
    
- File upload хийх
    
- Import status харах
    
- Import error харах
    

Харин mobile дээр bulk import-г primary action болгохгүй.

Primary action нь “нэг бараа хурдан нэмэх” байна.

---

# Responsive Component Standards

Бүх dashboard component дараах breakpoint зарчимтай байна.

## Mobile

- single column
    
- card layout
    
- bottom tabs
    
- bottom sheets
    
- sticky primary action
    
- large touch targets
    

## Tablet

- two column layout
    
- compact cards
    
- optional side panel
    

## Desktop

- sidebar
    
- tables
    
- full filters
    
- bulk actions
    

---

# Touch Target Rule

Mobile дээр бүх button, input, interactive item:

- хамгийн багадаа 44px өндөртэй
    
- spacing хангалттай
    
- нэг гараар дарахад амар
    

байна.

---

# Sticky Action Rule

Mobile form бүр дээр primary action sticky байна.

Жишээ:

Add Product page:

доод хэсэгт sticky:

- Cancel
    
- Save Product
    

байна.

---

# Keyboard Friendly Mobile Forms

Input бөглөх үед:

- зөв keyboard type ашиглана
    
- price талбар numeric keyboard нээнэ
    
- stock талбар numeric keyboard нээнэ
    
- barcode талбар scan хийх боломжтой байхаар бэлдэнэ
    
- Enter / Next дарахад дараагийн талбар руу шилжинэ
    

---

# PWA Readiness

Mobile ашиглалт чухал тул dashboard нь PWA байдлаар ажиллахад бэлэн байна.

Support хийх:

- add to home screen
    
- responsive manifest icons
    
- safe area inset
    
- standalone display mode
    
- offline fallback page
    
- slow network loading state
    

---

# Implementation Phases

## Phase 1: Mobile Navigation

- Dashboard mobile bottom tabs нэмэх
    
- Desktop sidebar-г mobile дээр нуух
    
- Mobile header нэмэх
    
- Store switcher mobile-д тохируулах
    

## Phase 2: Product Mobile UX

- Product list table-г mobile card болгох
    
- Add product page mobile-first болгох
    
- Quick stock edit нэмэх
    
- Quick price edit нэмэх
    
- Image upload UX сайжруулах
    

## Phase 3: Conversation Mobile UX

- Conversation list mobile card болгох
    
- Conversation detail mobile chat UI болгох
    
- AI/Human status ойлгомжтой харуулах
    

## Phase 4: Settings and AI Mobile UX

- AI settings-г card layout болгох
    
- Facebook settings mobile UX
    
- Billing mobile UX
    
- Members mobile UX
    

## Phase 5: Reports Mobile UX

- Dashboard summary cards
    
- Product insights cards
    
- Low stock cards
    
- AI usage cards
    

## Phase 6: PWA Polish

- Safe area support
    
- Install prompt
    
- Offline fallback
    
- Loading skeletons
    
- Mobile performance optimization
    

---

# Technical Notes

Use existing route structure as much as possible.

Do not create separate `/mobile` routes.

Instead:

- same route
    
- responsive components
    
- mobile-specific layout blocks
    
- desktop-specific table blocks
    

Example:

- desktop: product table
    
- mobile: product card list
    

Both use same data source.

---

# Acceptance Criteria

Mobile dashboard is considered complete when:

- хэрэглэгч утаснаасаа бараа нэмдэг
    
- зураг оруулдаг
    
- үнэ засдаг
    
- үлдэгдэл засдаг
    
- чат уншдаг
    
- AI тохиргоо өөрчилдөг
    
- төлбөрийн мэдээллээ хардаг
    
- Facebook холболтоо шалгадаг
    
- үндсэн тайлан хардаг
    

болсон байна.

---

# Golden Rule

Mobile дээр бүх боломж байх ёстой.

Гэхдээ бүх боломж нэг дор харагдах ёсгүй.

Үндсэн урсгал хурдан, нэмэлт боломжууд шаардлагатай үедээ гарч ирдэг байна.
## Progress
- [x] Created branch eature-mobile-first-dashboard in the repo.
- [x] Added mobile bottom tabs to dashboard layout.
- [x] Converted products page to mobile cards while keeping desktop table.
- [x] Converted conversations page to mobile cards while keeping desktop table.
- [ ] Remaining phases: settings, AI settings, billing, members, reports, import polish, PWA readiness.


## Progress Update 2
- [x] Mobile navigation added with bottom tabs.
- [x] Products page converted to mobile cards.
- [x] Conversations page converted to mobile cards.
- [x] Billing page converted to mobile cards and compact summary blocks.
- [x] Members page converted to mobile cards.
- [ ] Remaining work: AI settings, general settings polish, import flow, reports, PWA readiness, bottom-sheet quick actions.

