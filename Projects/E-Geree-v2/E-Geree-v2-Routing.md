---
title: "03 — Routing (App Router)"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v2
source_path: "D:/own/obsidian-vaults/E-Geree-v2-docs/03-Routing.md"
---

# Routing — Next.js App Router

[[E-Geree-v2-Home]] · өмнөх: [[E-Geree-v2-Architecture]] · дараах: [[E-Geree-v2-Contract-Create-Feature]]

Бүх route `src/app/[locale]/` дотор — олон хэлийг segment-ээр зохицуулна
(`mn` default / `en` / `ko` / `zh`). Route group-ууд `()` хаалтаар layout-г ялгана.
`src/middleware.js` нь locale routing (next-intl) + JWT шалгалтыг хариуцна.

## Route group-ууд
| Бүлэг | Auth | Layout |
|---|---|---|
| `(public)` | Үгүй | Минимал |
| `(protected)/(withSidebar)` | Тийм | Бүрэн sidebar shell |
| `(protected)/(without-sidebar)` | Тийм | Бүтэн өргөн |
| `(protected)/(manage)` | Тийм | Гэрээ үүсгэх wizard |
| `(verification)` | Хэсэгчилсэн | Баталгаажуулалтын UI |

## Хуудасны бүтэц (жишээ)
```
[locale]/
├─ (public)/            нээлттэй (blog, нийтийн гэрээ, pricing)
├─ (protected)/         нэвтэрсэн хэрэглэгч (middleware дээр шалгана)
│  ├─ (withSidebar)/    documents, archive, profile, team, notification
│  ├─ (without-sidebar)/  бүтэн өргөн хуудас
│  └─ (manage)/         contract create wizard  ← [[E-Geree-v2-Contract-Create-Feature]]
└─ (verification)/      хэрэглэгч/байгууллага баталгаажуулалт
```

## Дүрэм
- `page.jsx` файлууд **нимгэн** — зөвхөн `src/components/pages/<feature>/`-аас
  тохирох компонентыг import хийж render хийнэ.
- `.jsx` л байна — `.ts`/`.tsx` хориотой.
- Path alias: `@/` → `src/`.

Дэлгэрэнгүй context/state: [[E-Geree-v2-State-Management]]
