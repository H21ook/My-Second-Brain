---
title: "03 — Routing (App Router)"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/03-Routing.md"
---

# Routing — Next.js App Router

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-Architecture]] · дараах: [[E-Geree-v3-Contract-Create-Feature]]

Бүх route `src/app/[locale]/` дотор — олон хэлийг segment-ээр зохицуулна.
Route group-ууд `()` хаалтаар layout-г ялгана.

## Хуудасны бүтэц
```
[locale]/
├─ (auth)/            нэвтрэх бүлэг
│  ├─ login
│  └─ register
├─ (protected)/       нэвтэрсэн хэрэглэгч (server дээр шалгана)
│  ├─ (with-sidebar)/
│  │  ├─ documents/received
│  │  ├─ documents/sent
│  │  └─ profile
│  └─ (without-sidebar)/
│     ├─ contract/create/{participant,content,fields,submit}  ← wizard
│     └─ pdf-test
└─ (public)/          нээлттэй
   ├─ (home)
   └─ blog
```

## BFF route handlers (src/app/backend/)
- `auth/[...path]` — auth proxy (catch-all)
- `auth/{check-auth, login-by-email, logout, refresh-token}`
- `file/upload` — файл байршуулах
- `public-api/v1/[...path]`, `public-api/v2/[...path]` — гадаад API проксиг хувилбараар
- `health` — эрүүл мэндийн шалгалт

Дэлгэрэнгүй: [[E-Geree-v3-Networking-BFF]]
