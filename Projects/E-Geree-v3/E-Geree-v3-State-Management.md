---
title: "05 — Төлөв удирдлага (Redux)"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/05-State-Management.md"
---

# State Management — Redux Toolkit

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-Contract-Create-Feature]] · дараах: [[E-Geree-v3-Networking-BFF]]

Хоёр түвшний store:

## 1. Глобал store (`src/store/`)
- `auth-slice.ts` — `authSlice`, `AuthState` (нэвтрэлт, токен, профайл)
- `index.ts` — store тохиргоо
- Хандалт: `useAppSelector`, `useAppDispatch` (граф дээр хамгийн их холбоостой hook-ууд)

## 2. Feature store (`src/features/contract-create/store/`)
| Slice | Хариуцах хэсэг |
|---|---|
| `participants.slice.ts` | Гэрээний оролцогчид |
| `content.slice.ts` | Гэрээний агуулга, хугацаа |
| `fields.slice.ts` | PDF талбарууд |
| `meta.slice.ts` | Wizard meta (алхам, төлөв) |
| `selectors.ts` | Memoized selector-ууд (`makeSelectFieldsByPage` г.м.) |
| `persist.ts` | redux-persist тохиргоо (localStorage) |
| `pdf-storage.ts` | PDF файл хадгалалт |

## Persist
redux-persist-ээр wizard төлөв хадгалагдаж, хуудас дахин ачаалахад сэргээгдэнэ.
`useHydrated` hook нь SSR/hydration зөрүүнээс сэргийлнэ.

## PDF viewer store
PDF үзэгч өөрийн slice-тэй ([[E-Geree-v3-PDF-Viewer]]), `src/shared/pdf-viewer/store/`-д:
`pdf-viewer-slice.ts` (`pdfViewerSlice` — zoom/page), `pdf-field-slice.ts`
(`pdfFieldSlice` — талбарууд), `pdf-selectors.ts`. Store-н state key-ууд
(`pdfViewer`, `pdfField`) хэвээр.
