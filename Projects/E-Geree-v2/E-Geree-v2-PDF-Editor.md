---
title: "07 — PDF Editor"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v2
source_path: "D:/own/obsidian-vaults/E-Geree-v2-docs/07-PDF-Editor.md"
---

# PDF Contract Editor ба Field Editor

[[E-Geree-v2-Home]] · өмнөх: [[E-Geree-v2-Networking]]

`src/components/custom/pdf/` ба `src/components/context/pdf-context.jsx` —
төслийн **хамгийн нарийн** модуль. PDF дээр талбар (гарын үсэг, огноо, текст г.м.)
чирж байрлуулах overlay editor.

## Бүрэлдэхүүн хэсгүүд (граф-аас)
| Элемент | Үүрэг |
|---|---|
| `PDFEditorPage()` | Засварлах гол хуудас |
| `PDFViewerPage()` | Үзэх горим |
| `PDFPageCanvas` | Хуудас canvas дээр render |
| `DropableField()` | Чирж тавих талбар |
| `FieldsEdit` | Талбар засах төлөв |

## Координатын математик (ЧУХАЛ — бүү өөрчил)
Талбарууд page-relative PDF pixel-ээр хадгалагдана:
```
stored.left = (dropScreenX - canvasRect.left) / displayScale
displayScale = canvasDisplayWidth / canvasActualWidth
```
`PdfEditorPage.jsx`-ийн координатын тооцоог өөрчилбөл талбарууд чимээгүйхэн буруу
байрлана.

## Заавал баримтлах дүрэм
- Талбарын анхны хэмжээг **`getDefaultSize(fieldType)`** (PdfContext)-ээс авна —
  pixel хатуу бичихгүй.
- Нөхцөлд **`FieldType` enum** (`enums.js`) ашиглана — `"SIGNATURE"` гэх raw string биш.
- **`DndProvider` нэг л удаа** `src/app/[locale]/layout.jsx`-д (CustomProviders-ээр)
  mount хийгдсэн. Хоёр дахийг хаана ч нэмэхгүй — drag-and-drop чимээгүй эвдэрнэ.
- `pdf-context.jsx`-ийн `onResize` 16ms-ээр throttle хийгдсэн — энэ throttle-ийг
  бүү ав.

## Холбоо
Гэрээ үүсгэх wizard-ийн "Талбар" алхам ([[E-Geree-v2-Contract-Create-Feature]]) энэ editor-ийг
ашиглаж PDF дээр шууд талбар тавьдаг. Талбарын координат гэрээний төлөвт хадгалагдана
([[E-Geree-v2-State-Management]]).

## Remote зураг
PDF/зургийн remote host-уудыг `next.config.mjs`-ийн `remotePatterns`-д бүртгэнэ
(`cdn.e-geree.mn`, AWS S3, `api.qrserver.com`, `qpay.mn`, `s3.qpay.mn`).
