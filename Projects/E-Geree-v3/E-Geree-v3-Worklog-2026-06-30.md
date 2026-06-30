---
title: "E-Geree-v3-Worklog-2026-06-30"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/daily-notes/2026-06-30.md"
---
## /contract/create/content

1. pdf оруулчихсан байхад pdf input-н доорх алдааны текст арилахгүй байна.
2. toast-г арилгачих

## /contract/create/participants

1. Мэдээлэл бүрэн оруулна уу гэсэн toast харуулахаас аль оролцогчийн аль мэдээлэл оруулаагүй байгаа нь мэдэгдэхгүй байна


## /contract/create/fields

1. Оролцогч талуудын талбаруудыг харуулж байгаашдээ. Тэр ассordion-ий хамгийн дээр error хэсэг харуулчих юмуу. Тухайн тал дээр ямар алдаанууд байгааг
2. toast-г арилгах
### /contract/create/submit

1. Гарын үсэг, Огноо, Тамга талбар хүрээ нь улайж харагдмаар байна
2. Зурагны талбар зураг уншиж байх хооронд height нь 0 болоод байх шиг байна. min height тавиад loader уншуулах

---

## Хийгдсэн (2026-06-30, branch: feat/contract-create-rhf)

### /contract/create/content
- [x] **PDF оруулсны дараа алдаа арилахгүй** — PDF/таб солих нь native input биш (Redux→form синк) тул onChange валидаци ажиллахгүй байсан. `ContentStep` sync effect-д content алдаа байгаа бол `trigger("content")` дуудаж арилгана.
- [x] **toast арилгах** — `WizardShell` gate унах үеийн `validationFailed` toast устгав (бүх алхамд). Алдаа зөвхөн inline.

### /contract/create/participants
- [x] **toast → inline** — toast устгасан (дээрх). Participant алхам алдаатай оролцогчийг улаан ring + автомат сонголт + талбарын доорх текстээр (username/org/verification) аль хэдийн заадаг.

### /contract/create/fields
- [x] **Accordion дээд талд error хэсэг** — `ParticipantFieldsSidebar`: оролцогч бүрийн accordion content-ийн дээд талд улаан banner (AlertCircle + алдааны текст) ил гаргав. Hover tooltip хэвээр.
- [x] **toast арилгах** — дээрх WizardShell toast-аар хамт.

### /contract/create/submit
- [x] **Гарын үсэг/Огноо/Тамга улайх** — `SenderFillSection`: SignatureField/DatePickerField/ImageUploadField-д `invalid` prop.
- [x] **Зураг height 0 / loader** — ImageUploadField-д `min-h-24` + `imgLoading` state + onLoad/onError-оор Loader2 overlay.

Typecheck + ESLint цэвэр.

---

## Засвар 2 (participant алдаа огт харагдахгүй)

**Root cause:** `useFieldArray({ keyName: "participantKey" })` — keyName нь бодит дата талбартай мөргөлдөнө. RHF source: `fields.map((e,t)=>({...e,[keyName]: generatedId}))` → `fields[i].participantKey` нь RHF-ийн ДОТООД generated id болж дата утгыг дарж бичнэ. Тиймээс бүх `fields.findIndex(f => f.participantKey === realKey)` нь **-1** буцаж байсан → `hasError` үргэлж false, `usernameError`/`orgError` үргэлж undefined → алдаа хаана ч гарахгүй (handleRemove ч юу ч устгахгүй). Toast устгаснаар энэ эвдрэл ил болов.

**Fix (`ParticipantStep`):** `fields[].participantKey` lookup-уудыг устгаж, байрлалын индекс (participants ↔ errors.participants ↔ form values нэг дараалал)-ээр уншина:
- [x] errorKey → `participants[errorIndex]`
- [x] left-list hasError → `participantErrors[i]` (индекс)
- [x] ParticipantCard username/orgError → `participantErrors[selectedIndex]`
- [x] handleRemove → `participants.findIndex`
- [x] **Оролцогч тал улайх** — алдаатай мөр: улаан зүүн зураас + `bg-destructive/5` + AlertCircle икон
- [x] **Талбар дээр inline** — SectionContact username/org алдаа red border + текст (одоо lookup зассан тул ажиллана)

Typecheck + ESLint цэвэр.


