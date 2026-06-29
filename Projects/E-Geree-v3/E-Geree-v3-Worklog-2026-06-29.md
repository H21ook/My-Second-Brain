---
title: "E-Geree-v3-Worklog-2026-06-29"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/daily-notes/2026-06-29.md"
---

## /contract/create/participants

1. ✅ Default-р оролцогч 1 сонгогдсон байна
   - `ParticipantStep`: seed хийсний дараа `selectedKey` хоосон бол `useEffect`-ээр `participants[0]` оноох болгов
   - `useParticipantsSeed`: PRIVATE_CONTRACT үед participant1-д нэвтэрсэн хэрэглэгчийн профайл урьдчилан бөглөх болгов

### Мэдэгдэл хэсэг

1. ✅ Оролцогч 1 дээр email байх юм бол мэдэгдэл хэсгийг нуух
2. ✅ Бусад оролцогчдын username талбарт email ашигласан үед мөн адил нуух
3. ✅ Хувь хүн үед username дээр регситрын дугаар оруулвал notifyEmail бичих боломжтой
4. ✅ Байгууллага төлөөлсөн үед ажилтаны email бөглөөгүй бол notifyEmail бичих боломжтой
   → `SectionNotify`: `EMAIL_REG`-ээр username шалгаж имэйл бол `null` буцаана


## /contract/create/fields

1. ✅ Default-р оролцогч 1 сонгогдсон байна
   - `FieldsStep`: `effectiveParticipant` анхдагчийг `participants[1]` → `participants[0]` болгов
2. ✅ Оролцогч тал дээр Гарын үсэг зурах эрх тохируулсан бол Гарын үсэг талбар заавал шаардана. Аль оролцогч дээр гарын үсэг талбар тавиагүй гэдэг нь мэдэгдэхгүй байна.
   - `FieldsStep`: participant-key алдаа (needSignatureField гэх мэт) гарвал `activeParticipant` автоматаар тухайн оролцогч руу шилжнэ
   - `ParticipantFieldsSidebar`: алдаатай оролцогчийн accordion хил улаан болж `AlertCircle` icon харагдах, accordion автоматаар нээгдэнэ

### /contract/create/submit

1. ✅ Талбарын алдааг удирдах хэсэг React-Hook-Form ашиглаж байгаа шиг гоё ажиллахгүй байна
   - `SubmitStep.onUpdate`: талбарын утга өөрчлөгдөх үед тухайн талбарын алдааг errors state-аас шууд арилгана (RHF-style per-field clearing)
2. ✅ илгээсэн json data дотор contractFieldList дотор талбарын groupId орж ирэхгүй байна
   → `payload.ts` аль хэдийн `contractFieldList: fields` бүрэн дамжуулж байсан, groupId орно. autoDelete хасаж, contractType SENT/PUBLISHED тогтоов.