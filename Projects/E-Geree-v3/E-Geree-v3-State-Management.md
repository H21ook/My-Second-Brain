---
title: "05 — Төлөв удирдлага (Redux)"
type: project
status: active
created: 2026-06-30
updated: 2026-07-07
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/05-State-Management.md"
---

# State Management — Redux Toolkit

[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-Contract-Create-Feature]] · дараах: [[E-Geree-v3-Networking-BFF]]

Хоёр түвшний store. 2026-07-07 байдлаар root store 4 key-тэй хэвээр: `auth`, `pdfField`, `pdfViewer`, `contractCreate` (`src/store/index.ts:9-14`). Сүүлийн feature-ууд (documents detail, profile, label sidebar) шинэ slice нэмээгүй — тэдгээрийн server state бүхэлдээ TanStack Query-д; Redux нь client/wizard state-д л үлдсэн.

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
| `persist.ts` | Custom localStorage draft persist (redux-persist БИШ — доорх Persist хэсэг) |
| `pdf-storage.ts` | PDF файл хадгалалт |

## RHF ↔ Redux hybrid загвар (contract-create wizard)

2026-06-30-ны react-hook-form шилжилтээс хойш тогтсон, кодоор баталгаажсан (2026-07-07) загвар — feature дэлгэрэнгүй: [[E-Geree-v3-Contract-Create-Feature]], шилжилтийн түүх: [[E-Geree-v3-Worklog-2026-06-30]] (RHF plan устгагдсан 2026-07-07):

- **Алхам ДОТРОО** — per-step `useForm` + `zodResolver` нь source of truth. Schema factory-ууд createType-aware: `makeContentSchema` / `makeSubmitSchema` / `makeParticipantsSchema` (`src/features/contract-create/schema/index.ts:45,115,185`); хэрэглээ: `ContentStep.tsx:46`, `SubmitStep.tsx:100`, `ParticipantStep.tsx:92`.
- **Алхам ХООРОНД** — Redux (+persist) нь source of truth: алхам бүр тусдаа route тул шилжихэд step unmount болж RHF-ийн in-memory төлөв алга болно; амьд үлдэх нь Redux + localStorage draft + IndexedDB blob. Алхам mount дээр `defaultValues` ← Redux selector.
- **Гүүр** — `src/features/contract-create/hooks/useRhfReduxBridge.ts`: `form.watch` SUBSCRIPTION (callback хэлбэр, render-д утга ашиглахгүй → нэмэлт re-render үгүй) → **1000ms debounce** (default `delayMs`, мөр 26) → step бүрийн өгсөн Redux dispatch mapping. Доод давхаргад `persist.ts`-ийн 500ms throttle localStorage руу бичнэ.
- **Navigation gate (gate-registry)** — `src/features/contract-create/components/WizardStepGate.tsx`: идэвхтэй алхам mount дээрээ `useRegisterWizardGate(async () => trigger())` бүртгэнэ (`ContentStep.tsx:107`); `WizardShell.handleNext` нь `useWizardStepGateRunner()`-ээр дуудна; gate бүртгэлгүй бол `runGate()` нь `null` → блоклохгүй шилжинэ.
- **Онцгой тохиолдол** — FieldsStep canvas (drag/drop, undo) RHF биш, Redux хэвээр; gate нь `checkRequiredFields`-ээр шалгаж алдааг `fields.slice`-д тавьдаг (`FieldsStep.tsx:92`).

## Persist — custom subscriber (redux-persist БИШ)

Кодоор баталгаажсан (2026-07-07): wizard draft-ийг **redux-persist биш**, гар бичсэн custom subscriber хадгална — `package.json`-д redux-persist dependency огт байхгүй (Redux-ийн хамаарал нь зөвхөн `@reduxjs/toolkit`, `react-redux`).

- `src/features/contract-create/store/persist.ts` — `setupContractCreatePersistence()` (мөр 52): `store.subscribe` + **500ms throttle** (`THROTTLE_MS`, мөр 8), зөвхөн `contractCreate` subtree-г localStorage-ийн `CONTRACT_CREATE_DRAFT` key рүү бичнэ (мөр 7).
- `hydrateDraft()` (`persist.ts:87`) — draft уншиж `hydrateContractCreate` root action dispatch; server URL-гүй цэвэр local PDF байвал IndexedDB-ээс blob-ийг уншиж шинэ blob URL холбоно (`pdf-storage.ts`).
- Холболт: `src/features/contract-create/components/WizardBootstrap.tsx` — mount дээр эхлээд hydrate, дараа нь subscribe (анхны хоосон төлөв буцаж бичигдэхээс сэргийлнэ); unmount дээр flush хийгээд салгана.
- `useHydrated` hook (`src/hooks/useHydrated.ts`) нь SSR/hydration зөрүүнээс сэргийлнэ.

## PDF viewer store
PDF үзэгч өөрийн slice-тэй ([[E-Geree-v3-PDF-Viewer]]), `src/shared/pdf-viewer/store/`-д:
`pdf-viewer-slice.ts` (`pdfViewerSlice` — zoom/page), `pdf-field-slice.ts`
(`pdfFieldSlice` — талбарууд), `pdf-selectors.ts`. Store-н state key-ууд
(`pdfViewer`, `pdfField`) хэвээр.
