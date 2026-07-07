---
title: "07 — PDF Viewer"
type: project
status: active
created: 2026-06-30
updated: 2026-07-07
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
- `pdf-url.ts` — `resolvePdfUrl()`: CDN/cross-origin PDF URL-ийг same-origin proxy
  (`/pdf-api`, `/temporary`) зам руу хөрвүүлнэ (CORS). Анх `contract-create/lib`-д
  байсныг 2026-07-03 shared руу зөөсөн (2 feature ашигладаг болсон), `index.ts`-ээс
  export хийнэ.

## Хэрэглэгчид (consumers)
| Feature | Файл | Горим |
|---|---|---|
| contract-create | `FieldsStep.tsx` | `isEdit` талбар байрлуулах/чирэх |
| contract-create | `SubmitStep.tsx`, `PdfUploadField.tsx` | View-only (fields харуулах, edit-гүй) |
| documents (detail) | `ViewerPlaceholder.tsx` → `ContractDetail.tsx` | View-only + LIVE fields (local хуулбар, `FieldFillPanel`-аар засварлагдана), `signedPdfUrl \|\| generatedPdfUrl \|\| relatedPdfUrl` эх сурвалжаар (viewer: `e928bc5`; display: `43db7df` 2d-2a; fill: `b4ece39` 2d-2b). `isEdit`/`PdfEditConfig` хэрэггүй болсон — [[E-Geree-v3-Contract-Detail-Page-Phase2d-Plan]] 2d-2b. **Layout (`9e6409b`, дараа нь `c2c07e5`):** 2-col hero (`lg:grid-cols-[1fr_380px]`, `ContractDetail.tsx:279`), PDF = зүүн hero (`ViewerPlaceholder` `h-[60vh] lg:h-full`); баруун rail (Бөглөх/Мэдээлэл tabs) доор `ActionButtons` sticky rail-footer dock. `FlowActionDock` компонент устгагдсан (`c2c07e5`) — `ActionButtons.tsx`-д нэгтгэгдсэн: яг нэг primary товч (`getFlowAction` resolver, `lib/action-status.ts`) + бусад үйлдэл "⋯" overflow dropdown. Дэлгэрэнгүй: [[E-Geree-v3-Contract-Detail]]. |

## Талбар бөглөх UI (`src/shared/field-fill/`)
`SenderFillSection` — per-type value input widget (text/date/number/select/
signature-apply/image-upload). Анх `contract-create/components/steps/submit/`-д
байсныг 2026-07-03 энд зөөв (`boundaries/element-types` eslint дүрэм feature→
feature импортыг index.ts-ээр ч зөвшөөрдөггүй тул). `contract-create/SubmitStep.tsx`
(илгээгч, илгээхээс өмнө) болон `documents/FieldFillPanel.tsx` (хүлээн авагч,
SIGN_PENDING) хоёулаа хэрэглэнэ. Гарын үсэг зурах тусдаа UI хэрэггүй —
`auth.user.signatureImgUrl`-г шууд `value`-д apply хийдэг.

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
