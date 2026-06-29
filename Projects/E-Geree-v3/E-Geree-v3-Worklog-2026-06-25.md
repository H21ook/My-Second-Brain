---
title: "E-Geree-v3-Worklog-2026-06-25"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/daily-notes/2026-06-25.md"
---

## Гэрээ үүсгэх process засвар

### Алхам 1 (ContentStep)

URL: [http://localhost:3000/conract/create/content]

1. Гарчиг оруулаагүй үед алдааны мессеж Toast-р харагдаад байна. Энэ UX-д тохиромжтой эсэх. Хаана утга оруулагүй байгааг хайх хэрэгтэй болж байна.
2. PDF, Text төрөл солиход өмнө сонгосон байсан төрлийн контент арилаад байна. Энэ арилахгүй байвал зүгээр. Үргэлжлүүлэх товч дарахад сонгосон төрлийн контентийг үлдээгээд нөгөө төрлийн контентийг устгах
3. Нэмэлт тохиргоо хэсгийн switch-үүд default-р off байна.
4. И-мэйл гарчиг, И-мэйл агуулга inputs-н placeholder text засах
5. 

### Алхам 2 (ParticpantStep)

URL: [http://localhost:3000/conract/create/participant]

1. nickname талбарыг Хоч нэр биш Нэр гэж ашиглах
2. Хуулийн этгээд байгууллага сонгоод Байгууллагын регистр хэсэгт утга оруулаад хайх дараад байгууллага олдсон бол Зүүн талын Оролцогч item дээр 
	1. Байгууллагын нэр, регистр харуулах
	2. Гишүүний и-мэйл оруулаад Гишүүн шалгах товч заавал дарах (Байгууллага дээр тухайн и-мэйл, регистртэй гүшүүн байгаа эсэхийг шалгах) Шалгаад үр дүнг өнгөөр ялгах эсвэл текст өнгө хамт ашиглаж мэдэгдэх
	3. Гишүүний email оруулчихаад шалгаагүй бол алдаа өгөх ёстой шүү
	4. Гишүүний олдсон тохиолдолд Зүүн талын Оролцогч item дээр гишүүний и-мэйл, регистр харуулмаар байна. Гэхдээ яаж багтаах вэ?
3. Мэдэгдэл хэсгийг ард нь switch тавиад on болгожийж email оруулах input харуулах
	1. Дээр байгаа input нь түрүүлж нүдэнд тусахаар байх ёстой. Учир нь хүмүүс андуураад доорх талбар буюуу мэдэгдэл илгээх email талбарт гэрээ хүлээн авах email-г бичээд байдаг
4. Эрх ба баталгаажуулалт хэсэгт Эрх гэдэг label-г Баримт бичиг оролцох эрх болгоод өөрчлөөрэй. Эрхүүд доторх Талбар бөглөх гэдгийг Мэдээлэл бөглөх, Үзэх гэдгийг Харах болгох
5. Нууц үгээр хамгаалах хэсэгт идэвхжүүлээд нууц үг оруулсан. Тэгээд буцаагаа идэвхгүй болгосон бол оруулсан нууц үгийг цэвэрлэх
6. Энэ хуудаст алдааны мессежүүд алдаа гарч байгаа хэсгийг тодруулж хэрэглэгчид амархан алдаагаа олох тал дээр анхаарах 

### Алхам 3 (FieldsStep)

URL: [http://localhost:3000/conract/create/fields]

1. Оролцогч item-н title дээр хувь хүн үед nickname байвал харуулаад байхгүй бол  email/register харуулна. Байгууллага үед nickname | байгууллагын нэр, регистр ийм дарааллаар харуулна.
2. Талбар зөөгөөд хуудаст байршуулахад талбарын нэр нь default-р left sidebar хэсэгт харагдаж байгаа нэрээрээ байх
3. Left sidebar хэсэгт нэр нь бүтэн харагдахгүй байгаа талбарыг hover хийхэд tooltip нэрийг харуулах
4. Pdf хуудас дээр байршуулсан талбарын уртад нэр нь бүтэн харагдахгүй үед tooltip харуулах
5. Жишээ нь Оролцогч 1 дээр шинэ талбар тавихад аль хэдийн тавигдсан талбаруудын дарааллын index (order)-г reindex хийх. Дараалал буруу өсөхөөс сэргийлэх юм. Талбаруудын дараалал өөрчлөхгүйшдээ.
6. Хуудас дээрээс нэг талбар сонгосон. Гэрээний хуудасны талбаргүй хоосон газар дарахад focus алдаж байгаа. Харин хуудасны гадна буюу canvas background хэсэгт дарахаар focus алдахгүй байгаа юм. Энэ selected байхаа болих ёстой
7. Right sidebar дээр Оролцочийн талбарууд дотор Group талбар Accordion Нэр дээр дарахаар задрахгүйгээр эхний талбарыг select хийж байгаашдээ. Энийг дотроо олон талбаруудтай байвал задардаг байлгаж болох уу? Нэг талбартай байвал задардаггүй. Гэхдээ arrow дараад задарна. Ман group дотор олон талбар байвал accordion нэр дээр дахиад дарвал дараагийн талбарыг select хийх. Нэр дээр дараад group доторх талбаруудыг cолих боломжтой гэсэн үг.
8. Right sidebar дээрээс талбар сонгоход pdf дээр талбарын байгаа газарлуу scroll-дож олж очих
9. Right sidebar дээр Оролцогч 1-ийн талбарыг сонгосон бол Оролцогц 1-ээс бусад accordion хаагдах
10. Right sidebar дээрээс талбаруудыг drag and drop order хийдэг байх (order index солино)
11. Талбарын тохиргоо хэсгийн input-уудын утгыг хязгаарлах (Жишээ хуудасны x,y-с хэтрүүлэх, pdf-н хуудасны тооноос их эсвэл бага хуудасны дугаар тохируулах гэх мэт) алдаанаас сэргийлэх
12. Сонголт талбар байрлуулчихаад сонгох утга оруулаагүй үед алдаа өгөх. Заавал утга оруулах ёстой.
13. System type-тай талбаруудын нэрийг өөрчлөх боломжгүй байх


### Алхам 4 (SubmitStep)

URL: [http://localhost:3000/conract/create/submit]

1. Left sidebar дээр Оролцогчийн мэдээлэл дээр Accordion Trigger дээрх title нь nickname | Илгээгч/Хүлээн авагч, харин доорх жижиг текст нь иргэн үед email/register, байгууллаг үед байгууллагын нэр, регситр харуулах
2. Left sidebar дээр Оролцогчийн мэдээлэл дотор Баталгаажуулалтын мэдээлийн доор Нууц үг хийсэн бол Нууц үг идэвхитэй гэж харуулах. Trigger дээр байгаа lock icon байж байг
3. Right sidebar дээр бөглөж байгаа талбарын pdf хуудас дээрх талбарлуу scroll focus-лаж очих, Select, Гарын үсэг, Зураг, Тамга, Огноо талбаруудын onChange дээр талбарлуу очих, Текст, Тоо талбаруудын focus дээр талбарлуу очих
4. Required талбаруудыг бөглөөгүй Гэрээ илгээх дарвал алдааны мессеж харуулах хаана яагаад алдаа өгч байгааг тодорхой харуулах
5. Гэрээ илгээх дарахад баталгаажуулах modal гарч ирж байгаашдээ. Тэндээс зөвшөөрөх дарах үед файл нь хуулахад алдаа гарчихвал очоод файлаа солих боломжийг хэрэглэгчид амархан өгөх зориготой file upload хийх, гэрээ илгээх 2г step loader маягаар харуулах санаа төрсөн юм. Чи хэрэггүй гэвэл шаардлаггүй.  

### Ерөнхий тэмдэглэл

Талбаруудын алдаа, value, changes-г хэрхэн control-дох аргыг тодорхойлох. react-hook-form ашиглах эсэх (Манай баг react-hook-form туршлагатай)

Гэрээ үүсгэх бүтэн process ингээд болж байгаа учир, гэрээ үүсгэхэд цуглуулсан дата indexDb зэрэг датагаа яаж цэвэрлэх, хэзээ цэвэрлэх гэх мэт flow-г тодорхойлох. vault-д features/create-contract.md анхаарах зүйлс, state хамаарал гэх мэт тэмдэглээд өгөөрэй.

Дээрх өөрчлөлтүүдийг хийхдээ энэ файлд явцыг тэмдэглээрэй. dev-khishigee branch-с branch үүсгээд хийгээрэй

---

## Явц (Claude)

**Branch:** `feat/contract-create-ux-2026-06-25` (dev-khishigee-с салаалсан)

**Шийдвэр:** Form/error control-г одоогийн Redux-аар үргэлжлүүлэв (react-hook-form-г дараа тусдаа ажил болгоно). Item-уудыг Алхам 1-ээс дараалан, step тус бүрд commit.

### Алхам 1 (ContentStep) — ✅ Дууссан
- [x] **1.** Toast хэвээр үлдээгээд, нэмж **inline алдаа** оруулав. Гарчиг/контент талбарын дор тухайн алдааг шууд харуулна (`aria-invalid` + улаан текст). Шинэ дэд бүтэц: `validateStep` нь `errors: {fieldKey→i18nKey}` буцаадаг болов; `WizardShell` дотор `WizardErrorsProvider` context (transient, draft-д хадгалагдахгүй); талбарууд `useFieldError(key)`-ээр уншина. → Алхам 2,3,4-т дахин ашиглана.
- [x] **2.** Төрөл солиход контент УСТГАХГҮЙ болов. PDF/Text-ийн аль алинд оруулсан дата хадгалагдана; "Үргэлжлүүлэх" дарахад л сонгоогүй төрлийн контентийг `WizardShell.handleNext` цэвэрлэнэ (TEXT сонгосон бол PDF + IndexedDB blob, PDF сонгосон бол текст).
- [x] **3.** Нэмэлт тохиргооны switch default off: `content.slice` дахь `showQrCode: true → false` (autoDelete аль хэдийн false).
- [x] **4.** mn.json placeholder засав: "Гэрээний сэдэв/тайлбар" → "И-мэйлийн гарчиг/агуулга" (label-тай тааруулав). en/cn/kr нь аль хэдийн ерөнхий ("Enter the subject") тул хэвээр.

**Хөндсөн файлууд:** `lib/validation-steps.ts`, `components/wizard-errors.ts` (шинэ), `components/WizardShell.tsx`, `components/steps/ContentStep.tsx`, `store/content.slice.ts`, `i18n/languages-data/mn.json`.

### Алхам 2 (ParticipantStep) — ✅ Дууссан
- [x] **1.** `nickname` label "Хоч нэр" → "Нэр" (mn.json). _en/cn/kr-д ижил relabel хийх шаардлагатай._
- [x] **2.** Байгууллагын гишүүн шалгах урсгал (`useCompanyEmployeeCheck`). SectionContact org салбарт гишүүний и-мэйл/регистр input + "Гишүүн шалгах" товч; олдвол `companyEmployeeId` хадгалж ногоон "Гишүүн олдлоо" харуулна, олдохгүй/буруу бол улаан алдаа. Регистр/гишүүн солиход баталгаажуулалт цэвэрлэгдэж дахин шалгуулна. **2.3:** гишүүн оруулчихаад шалгаагүй бол валидаци `checkMemberRequired` алдаа өгнө. **2.1/2.4:** зүүн item — 2 мөр: гарчиг = nickname→байгууллагын нэр→дүр, дэд мөр = регистр · гишүүний username (иргэн бол username). Шинэ key-ууд: `checkMember`, `memberFound`, `memberNotFound`, `checkMemberRequired` (4 локаль).
- [x] **3.** Мэдэгдэл хэсэгт switch нэмэв (default OFF). Идэвхжүүлж байж и-мэйл input гарна; унтраахад хаягийг цэвэрлэнэ. Шинэ key `notifyEnable` (4 локальд). Default OFF учир гэрээ хүлээн авах и-мэйлтэй андуурахаас сэргийлж байгаа (3.1).
- [x] **4.** Permission label-ууд (mn.json): "Эрх" → "Баримт бичигт оролцох эрх"; "Талбар бөглөх" → "Мэдээлэл бөглөх"; "Үзэх" → "Харах". _en/cn/kr-д ижил relabel хийх шаардлагатай._
- [x] **5.** Нууц үг — **аль хэдийн хэрэгжсэн байсан**: `SectionSecurity`-д switch унтраахад `accessCode: null` болгодог (онцлох өөрчлөлт шаардлагагүй).
- [x] **6.** Алхмын алдааг inline тодруулав. `validateParticipant` одоо `${participantKey}:field` түлхүүртэй errors map буцаана; `WizardErrorsProvider` (Алхам 1-ийн дэд бүтэц) дамжуулна. `useWizardErrors` нэмж — валидаци унахад алдаатай оролцогчийг **автоматаар сонгоно** (render үед, effect-гүй); талбарууд (org регистр, гишүүн/иргэний username) `useFieldError`-оор дороо улаан алдаа харуулна.

**Хөндсөн файлууд (Алхам 2):** `lib/validation-steps.ts`, `components/wizard-errors.ts`, `components/steps/ParticipantStep.tsx`, `components/steps/participant/SectionNotify.tsx` + `SectionContact.tsx`, 4 локалийн json.

### Алхам 3 (FieldsStep) — ✅ Дууссан (13/13 item)
Бүх item хийгдсэн. Typecheck + eslint цэвэр, vitest 24/24 ногоон. Commit-ууд:

- **Багц A (#1,2,3,4,8,11,13):**
  - **#1** `participantDisplayParts()` shared helper (`lib/participants.ts`) — хувь хүн: nickname/username; байгууллага: nickname/орг нэр. FieldsSidebar + ParticipantFieldsSidebar-т хэрэглэв. _(ParticipantStep-ийн ажиллаж буй inline хувилбарыг хөндөөгүй.)_
  - **#2** Тавьсан талбарын нэр default = sidebar label (`handleDragStart` payload-д `name` ононо).
  - **#3** Зүүн самбарын таслагдсан талбар/оролцогч нэрд `title` tooltip.
  - **#4** PDF дээрх талбарын нэрд `title` tooltip (`renderField`).
  - **#8** Баруун самбараас талбар сонгоход PDF хуудас руу `scrollToPage` (scroll-context дахин ашиглав).
  - **#11** X/Y input-д хуудасны хязгаар `max` (commit дээр clamp аль хэдийн байсан).
  - **#13** SYSTEM/COMPANY/DAN талбарын нэр `disabled` (засах боломжгүй).
- **#9** Баруун самбарын accordion controlled — талбар сонгоход тухайн оролцогчоос бусад нь хаагдана (`handleSelect`-д `setOpenKeys`, effect-гүй).
- **#7** Олон гишүүнт group-ийн нэр дээр дарвал задлаад дараагийн гишүүн рүү цикл (`handleNameClick`); ганц гишүүнт зөвхөн select, arrow задлана.
- **#5** Шинэ талбар тавихад `addField` reducer тухайн оролцогчийн `order`-г 1..n болгож reindex (харьцангуй дараалал хэвээр). Тест: `store/fields.slice.test.ts`.
- **#12** Сонголтгүй SELECT талбар → inline алдаа. `checkRequiredFields` `select:${id}`-ээр key, `validateFields` errors буцаана, `FieldSettingsPanel`→`SelectOptionsEditor` харуулна, `FieldsStep` алдаатай талбарыг авто-сонгоно.
- **#6** Хуудасны гадна canvas фон дээр дарахад ч сонголт цэвэрлэгдэнэ (FieldsStep wrapper onClick, `[data-field-id]`-ээр талбар ялгана — shared viewer хөндөөгүй).
- **#10** Баруун самбарт group блокуудыг grip-ээс HTML5 drag-drop-оор эрэмбэлнэ. `reorderParticipantGroups` slice action (order 1..n). Тест нэмсэн.

**Хөндсөн файлууд:** `lib/participants.ts`, `lib/fields.ts`(+test), `lib/validation-steps.ts`, `store/fields.slice.ts`(+test), `components/steps/FieldsStep.tsx`, `steps/fields/{FieldsSidebar,ParticipantFieldsSidebar,FieldSettingsPanel,SelectOptionsEditor}.tsx`.

**Үлдсэн tech-debt:** байгууллага оролцогчийн регистрийг самбарын subline slot permission/fieldCount эзэлсэн тул title-д шахаагүй (#1 — ParticipantStep-д бүтнээр харагдана). `needSelectOption` 4 локалд бүгд бий.

### Алхам 4 (SubmitStep) — ✅ Дууссан (item 1–4; item 5 алгассан)
- [x] **1.** `ReviewSection` accordion: гарчиг = nickname → (Илгээгч/Хүлээн авагч), дэд мөр = иргэн бол username (email/register), байгууллага бол "нэр · регистр". (shared `participantDisplayParts` нь org-нэрийг primary-д тавьдаг тул энд spec-ийн дагуу inline тооцоолов — helper-ийг хөндөөгүй.)
- [x] **2.** Нууц үгээр хамгаалсан config-д AccordionContent дотор "Нууц үг идэвхтэй" мөр + Lock icon (trigger дээрх lock хэвээр). Шинэ түлхүүр `passwordActive` (4 локал).
- [x] **3.** Баруун самбараас бөглөхөд PDF хуудас руу scroll. `viewerRef`+`scrollToPage` SubmitStep-д (FieldsStep-тэй ижил handle). SELECT/SIGNATURE/IMAGE/STAMP/DATE — onChange дээр, TEXT/NUMBER — onFocus дээр гүйлгэнэ.
- [x] **4.** "Гэрээ үүсгэх" дарахад илгээгчийн required талбар бөглөгдөөгүй бол inline алдаа (`fieldRequired` дахин ашиглав) + эхний алдаатай талбар руу scroll + toast; AlertDialog controlled (`confirmOpen`) болгож, валид бол л нээнэ. `aria-invalid` талбаруудад.
- [ ] **5.** File upload алдаа → дахин солих + step loader — **алгассан** (note-д "заавал биш"). Одоогийн `handleSubmit` upload алдааг catch + toast хийдэг хэвээр.

**Хөндсөн файлууд:** `components/steps/SubmitStep.tsx`, `steps/submit/{SenderFillSection,ReviewSection}.tsx`, 4 локалийн json. Typecheck + eslint цэвэр.