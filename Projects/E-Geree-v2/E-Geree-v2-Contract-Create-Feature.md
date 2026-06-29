---
title: "04 — Гэрээ үүсгэх feature"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v2
source_path: "D:/own/obsidian-vaults/E-Geree-v2-docs/04-Contract-Create-Feature.md"
---

# Contract Create — гол feature

[[E-Geree-v2-Home]] · өмнөх: [[E-Geree-v2-Routing]] · дараах: [[E-Geree-v2-State-Management]]

`(protected)/(manage)` route group дотор амьдардаг, олон алхамт wizard. Компонентууд
`src/components/pages/`-д, төлөв нь `useContractState()` context-д хадгалагдана.

## Гол хэсгүүд (граф-аас)
| Элемент | Үүрэг |
|---|---|
| `useContractState()` | Wizard-ийн төлөв удирдах context hook (33 edge) |
| `CreateFiveMinutes()` | Хурдан гэрээ үүсгэх урсгал |
| `ManageContentSidebar()` / `ContentSidebar()` | Агуулгын хажуу самбар |
| `SignTemplateContent()` | Гарын үсгийн template агуулга |
| `SubmitPageContent()` | Эцсийн илгээх алхам |
| `DocumentType()` | Баримтын төрөл сонгох |

## Wizard урсгал
```
Оролцогч → Агуулга → Талбар (Fields) → Илгээх (Submit)
```
1. **Оролцогч** — гэрээний оролцогчид (хувь хүн / хуулийн этгээд)
2. **Агуулга** — гэрээний агуулга, нөхцөл, хугацаа
3. **Талбар** — PDF дээр талбар (гарын үсэг, текст г.м.) байрлуулна → [[E-Geree-v2-PDF-Editor]]
4. **Илгээх** — Server Action-аар service дуудаж backend руу илгээнэ → [[E-Geree-v2-Networking]]

## Холбогдох талбар (field) хэсэг
- `FieldGroupEntity()`, `FieldListEntity()`, `FieldList()` — талбарын модель
- `getFieldTypeText()`, `fontSizeOptionList` — талбарын тохиргоо
- `ParticipantFieldSidebar()` — оролцогч бүрийн талбарын самбар
- `FieldType` enum (`src/utils/enums.js`) — талбарын төрлүүд

## Domain enum-ууд
`ContractStatusEnum`, `ContractType`, `CREATE_TYPE`, `CompanyEmployeeRole`,
`CancellationPartyType` — бүгд `src/utils/enums.js`-д. Шинэ тогтмол нэмэхээсээ өмнө
эндээс шалгана.
