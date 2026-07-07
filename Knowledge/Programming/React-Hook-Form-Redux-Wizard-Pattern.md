---
title: "React-Hook-Form-Redux-Wizard-Pattern"
type: knowledge
status: active
created: 2026-07-07
updated: 2026-07-07
tags:
  - knowledge
  - programming
  - react
  - react-hook-form
  - redux
  - forms
---

# React Hook Form + Redux Wizard Pattern

## Summary

Олон алхамт wizard-ийн алхам бүр **тусдаа route** (алхам солиход component unmount болдог) үед react-hook-form-ыг Redux-тэй хослуулах загвар: алхам бүрд өөрийн `useForm`, Redux нь алхам хоорондын source of truth, хооронд нь `watch` subscription → debounce → `dispatch` гүүрээр холбоно. Navigation validation-ыг gate-registry context-ээр шийднэ.

## Problem

RHF-ийн state нь in-memory тул алхам = route байх үед navigation бүрд устна. Single mega-form хийе гэвэл route-ууд тусдаа mount/unmount болдог, мөн canvas маягийн (drag/drop, undo) алхам RHF-д таардаггүй. Харин бүх input-ыг Redux-аар шууд удирдвал keystroke бүрд dispatch + re-render, per-field validation/`touched`/`dirty` алга, error төлөв олон газар тарна.

## Core Concepts

- **Per-step `useForm`** — алхам бүр өөрийн form; `defaultValues` ← Redux selector (өмнөх бөглөлт unmount-ыг давсан тул сэргэнэ). Дэд хэсэгт `FormProvider` + `useFormContext`.
- **Хоёр давхар source of truth** — RHF = алхам ДОТОРХ (засвар, per-field validate/error); Redux = алхам ХООРОНДЫН (persist: localStorage draft, IndexedDB blob). Redux-ийн persist давхаргад гар хүрэхгүй.
- **Hybrid exception** — form input биш interaction (canvas, drag/drop, undo) Redux хэвээр; зөвхөн түүний settings panel-ийн input RHF.
- **Gate-registry navigation** — shell дэх "Next" товч алхмын validation-ыг мэдэхгүй; идэвхтэй алхам өөрөө gate бүртгэнэ.

## How It Works

### 1. RHF → Redux гүүр (watch subscription → debounce → dispatch)

```ts
export function useRhfReduxBridge<TValues extends FieldValues>(
  form: UseFormReturn<TValues>,
  onChange: (values: TValues) => void,
  delayMs = 1000,
): void {
  const onChangeRef = useRef(onChange);
  useEffect(() => { onChangeRef.current = onChange; }, [onChange]);

  const { watch } = form;
  useEffect(() => {
    let timer: ReturnType<typeof setTimeout> | undefined;
    const subscription = watch((values) => {
      if (timer) clearTimeout(timer);
      timer = setTimeout(() => onChangeRef.current(values as TValues), delayMs);
    });
    return () => { if (timer) clearTimeout(timer); subscription.unsubscribe(); };
  }, [watch, delayMs]);
}
```

Гол дүрмүүд:

- **`watch()`-ийн буцах утгыг render-д ХЭЗЭЭ Ч ашиглахгүй** — subscription (callback) хэлбэрээр л. Ингэвэл RHF-ийн uncontrolled input-ууд keystroke бүрд re-render үүсгэхгүй.
- **Debounce (~1000ms)** — sync нь зөвхөн draft persist-д хэрэгтэй тул хурдан байх шаардлагагүй; доод persist давхаргын өөрийн throttle-тэй (жишээ нь 500ms) уялдана.
- **`onChange`-ыг ref-ээр дуудах** — callback өөрчлөгдөх бүрд subscription дахин үүсгэхгүй.

### 2. Gate-registry navigation

Жижиг context: идэвхтэй алхам mount дээрээ async validator бүртгэнэ, shell "Next" дээр түүнийг дуудна.

```ts
// Алхам дотор:
useRegisterWizardGate(async () => form.trigger());

// Shell-ийн handleNext:
const ok = await runActiveGate();
if (!ok) return;
goNext();
```

- Алдаа inline нь RHF `formState.errors`-оор гарна — `form.trigger()` error-уудыг талбар бүрд автоматаар тавьдаг.
- Async server-check (жишээ нь гишүүнчлэл шалгах) sync Zod resolver-д багтахгүй → үр дүнг ref-д хадгалж gate = `trigger() && asyncOk`. Анхаар: `form.trigger()` нь `setError`-оор гараар тавьсан алдааг арилгадаг тул gate дотор async алдааг trigger-ийн ДАРАА дахин тавина (resync).
- Terminal (submit) алхам gate ашиглахгүй — өөрийн `handleSubmit(onValid, onInvalid)`.

### 3. Validation-ы нэг эх сурвалж

Гараар бичсэн `validateStep()` функцүүдийг Zod schema factory болгож `zodResolver`-т өгнө; нөхцөлт (mode-aware) дүрмүүдийг schema factory-ийн параметрээр салгана. Хуучин validation module + тусдаа error context-уудыг устгаж бүх алдааг `formState.errors` нэг загварт оруулна.

## useFieldArray `keyName` pitfall

`useFieldArray({ keyName: "myKey" })` үед RHF нь `fields[i].myKey`-г **өөрийн generated id-аар ДАРЖ бичдэг**:

```ts
// RHF source (үзэл баримтлал):
fields.map((e) => ({ ...e, [keyName]: generatedId }))
```

Хэрэв `keyName` нь өгөгдлийн БОДИТ талбарын нэртэй давхцвал (жишээ нь `participantKey`) тухайн талбараар хийсэн бүх lookup чимээгүй унана — `fields.findIndex(f => f.myKey === realKey)` үргэлж `-1`, алдаа хаана ч харагдахгүй, remove ч буруу ажиллана.

Хамгаалалт:

- `fields`, `form values`, `formState.errors` массивууд **нэг дараалалтай** тул lookup-ыг key-ээр биш **index тэгшитгэлээр** хийх; эсвэл
- бодит утга хэрэгтэй бол `form.getValues("items")[i]`-ээс унших (fields snapshot-оос биш); эсвэл
- `keyName`-д дата талбартай давхцахгүй нэр өгөх.

(RHF v8-д `keyName` deprecated — гэхдээ default `id` ч мөн адил дата дахь `id` талбарыг дардаг тул зарчим хэвээр.)

## When to Use

- Wizard-ийн алхам бүр тусдаа route/page бөгөөд draft persistence (refresh давах) шаардлагатай.
- Form ergonomics (per-field validation, touched/dirty) + global state (persist, cross-step derived data) хоёулаа хэрэгтэй.

## When Not to Use

- Нэг route доторх wizard — нэг `useForm` mounted үлдээхэд л хангалттай, Redux гүүр илүүц.
- Draft хадгалах шаардлагагүй богино form — RHF дангаараа.
- Canvas-маягийн чөлөөт interaction — RHF биш, шууд store.

## Common Mistakes

- `form.watch()`-ыг render дотор дуудаж утгыг нь ашиглах → бүх form re-render (subscription хэлбэрийг ашигла; targeted хэрэгцээнд `useWatch`).
- Debounce-гүй dispatch → keystroke бүрд store update, persist давхарга шуугина.
- `useFieldArray` keyName ↔ дата талбарын мөргөлдөөн (дээрх хэсэг).
- Шилжилтийн үед зарим алхам RHF, зарим хуучин загвартаа үлдэх → давхар source of truth; алхам бүрийг бүрэн дуусгаж merge хийх.
- Gate дотор async алдааг `trigger()`-ээс өмнө тавих → trigger арилгачихна.

## Trade-offs

- **+** Per-field UX (onBlur validate, талбар засахад алдаа арилах), uncontrolled input тул хуучин "keystroke бүрд dispatch"-аас хөнгөн, persist давхарга өөрчлөгдөхгүй.
- **−** Хоёр давхар state-ийн уялдааг (bridge timing, defaultValues seed) ойлгож байх шаардлага; debounce-ийн хэмжээнд crash үед сүүлийн хэдэн секундын өөрчлөлт алдагдаж болно.

## Examples

Бодит хэрэгжүүлэлт: [[E-Geree-v3-Contract-Create-Feature]] —

- `src/features/contract-create/hooks/useRhfReduxBridge.ts` (гүүр hook, дээрх код),
- `src/features/contract-create/components/WizardStepGate.tsx` (gate-registry context),
- `src/features/contract-create/components/steps/ParticipantStep.tsx:99-105, 230-233` (`keyName: "participantKey"` + index-alignment workaround; root cause-ийн шинжилгээ [[E-Geree-v3-Worklog-2026-06-30]]).

## Related Notes

- [[E-Geree-v3-Contract-Create-Feature]] — wizard-ийн бүтэц, persistence, cleanup flow.
- [[E-Geree-v3-State-Management]] — Redux slice-уудын зохион байгуулалт.
