---
title: "07 — PDF Viewer"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/07-PDF-Viewer.md"
---

# PDF Viewer ба Field Editor

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-Networking-BFF]]

`src/shared/pdf-viewer/` — pdfjs-dist v5 дээр суурилсан PDF үзэгч,
дээр нь талбар (field) байрлуулах overlay editor. Дундын (shared) давхаргад
байрлах ба contract-create feature түүнийг ашигладаг.

## Бүрэлдэхүүн хэсгүүд
| Файл | Үүрэг |
|---|---|
| `PdfViewer.tsx` | Гол үзэгч компонент |
| `PdfViewerPage.tsx` / `PdfPage.tsx` | Хуудас render |
| `PdfPageCanvas.tsx` | Canvas дээр зурах |
| `PdfEditorPage.tsx` | Засварлах горим |
| `FieldsOverlay.tsx` | Талбаруудыг харуулах давхарга |
| `FieldsEditOverlay.tsx` | Талбар чирэх/засах давхарга |
| `types.ts` | `PageDimension`, талбарын төрлүүд |

## Hooks (`hooks/`)
- `usePdfDocument` — PDF баримт ачаалах (`UsePdfDocumentResult`)
- `usePdfViewport` — zoom/viewport математик (`clamp`, `getTouchCenter`, `getTouchDistance`)
- `usePdfVisibility` — харагдах хуудас илрүүлэх

## Store (`store/`)
- `pdf-viewer-slice.ts` — zoom, идэвхтэй хуудас (`pdfViewerSlice`)
- `pdf-field-slice.ts` — талбарын төлөв (`pdfFieldSlice`)
- `pdf-selectors.ts` — selector-ууд

## Utils (`utils/`)
- `pdfjs-loader.ts` — pdfjs worker ачаалах
- `pdf-field-utils.ts` — талбар координат тооцоо

## Талбар сонголтын animation (`FieldsEditOverlay`)
- **Бүдгэрэлт** — талбар сонгогдсон үед (`selectedFieldId`) сонгосноос бусад нь
  `opacity 0.35` болж бүдгэрнэ (бүх тохиолдолд: sidebar, PDF click, шинэ талбар).
- **Pop** — зөвхөн sidebar-аас scroll-аар сонгоход (`PdfEditConfig.pulseTick`
  тоолуур нэмэгдэхэд) тухайн box нь scroll очсоны дараа (~450ms) `scale 1→1.08→1`
  pop хийнэ. PDF дээр шууд дарах / шинэ талбар тавихад `pulseTick` хэвээр тул
  pop-гүй, instant.
- **`pulseTick` эх сурвалж** — `FieldsStep` нь scroll-context-оор `scrollAndPulse`
  өгч, sidebar талбар сонгоход scroll + `pulseTick++`. Keyboard хуудас солих нь
  raw `scrollToPage` (pulse-гүй). Overlay нь `popped.id === selectedFieldId`
  тэнцвэл л focus идэвхтэй — PDF дээр өөр талбар дармагц автоматаар унтарна.

## Холбоо
Fields wizard алхам ([[E-Geree-v3-Contract-Create-Feature]]) энэ үзэгчийг ашиглаж
PDF дээр шууд талбар тавьдаг. Талбарын координат feature store-д хадгалагдана
([[E-Geree-v3-State-Management]]).
