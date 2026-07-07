---
title: "00 — Тэмдэглэл хөтлөх заавар"
type: project
status: draft
created: 2026-06-30
updated: 2026-07-07
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/00-Note-Keeping-Guide.md"
---

# Тэмдэглэл хөтлөх заавар (Note-keeping)

[[E-Geree-v3-Home]]

Энэ vault-ийг жигд хөтлөх конвенц. **Хүн (би) болон AI session (Claude/Codex зэрэг AI агентууд) хоёулаа** үүнийг дагана — шинэ тэмдэглэл бичих, distill хийх, цэвэрлэх бүгд энэ дүрмээр.

Энэхүү vault нь **төслийн урт хугацааны мэдлэгийн сан (Knowledge Base)** бөгөөд production кодын хуулбар биш. Production code нь source of truth; vault нь түүний distill хийсэн тогтмол мэдлэг.

---

## Үндсэн зарчим

- **Plan** → Хэрхэн хийх вэ?
- **Worklog (daily note)** → Юу болсон бэ?
- **Reference** → Цаашид юу үнэн хэвээр үлдэх вэ?

AI нь үргэлж эдгээрийг ялгаж ойлгоно.

---

## Note-ийн 3 үндсэн төрөл

Бүх note **энэ фолдерт flat** байрлана — дэд фолдер (`plans/`, `daily-notes/`), `NN-` дугаарлалт байхгүй.

| Төрөл                    | Нэр (flat, энэ фолдерт)                                                                                   | Frontmatter                                          | Хэзээ ашиглах                                  | Амьдралын мөчлөг                                   |
| ------------------------ | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------- | -------------------------------------------------- |
| **Reference**            | `E-Geree-v3-<Purpose>.md` (`E-Geree-v3-Overview`, `E-Geree-v3-Contract-Create-Feature`) — [[E-Geree-v3-Home]]-д бүртгэнэ | `tags`-д `project/e-geree-v3`                        | Тогтмол үнэн — архитектур, feature, дэд систем | Удаан амьдарна; шинэчилж байна, устгахгүй          |
| **Worklog (daily note)** | `E-Geree-v3-Worklog-YYYY-MM-DD.md`                                                                          | (заавал биш)                                         | Өдрийн ажил, явц, түр санаа, debug             | Богино настай — дуусахад reference рүү **distill** |
| **Plan (устгаж болох)**  | `E-Geree-v3-<Name>-Plan.md`                                                                                 | `tags`-д `plan` (эсвэл `egeree/plan`) + `status: proposed\|in-progress\|done` | Хэрэгжүүлэх төлөвлөгөө/proposal                | Хэрэгжиж, distill хийгдсэний дараа **устгана**     |

---

## Reference (`E-Geree-v3-<Purpose>.md`)

- **Нэр:** `E-Geree-v3-<Purpose>.md` — flat, дугааргүй. Сэдвийн логик дарааллыг файлын нэр биш,
  [[E-Geree-v3-Home]]-ийн бүртгэлийн жагсаалт тодорхойлно
  ([[E-Geree-v3-Overview]] → [[E-Geree-v3-Architecture]] → …).
- **Frontmatter:** одоо байгаа reference-уудтай ижил — `title` (Монгол), `type: project`,
  `status`, `created`, `updated`, `tags`-д `project/e-geree-v3`.
- **Breadcrumb (эхний мөрөнд):** `[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-…]] · дараах: [[E-Geree-v3-…]]`,
  шаардвал `· холбоотой: [[…]]`.
- **Шинэ reference нэмбэл** [[E-Geree-v3-Home]]-д бүртгэх; өмнөх/дараах breadcrumb-ийг хөршүүдэд нь холбох.
- **Тогтмол үнэн байх** — өдрийн явц/ноорог энд орохгүй (тэр нь worklog/plan).

> [!important] **Кодын бүтэц/логик өөрчлөгдвөл холбогдох reference-г заавал шинэчлэх.**
> Жишээ: contract-create store/flow өөрчилбөл `[[E-Geree-v3-Contract-Create-Feature]]` /
> `[[E-Geree-v3-State-Management]]`-г update хийх. Reference шинэчлэх нь ажлын "дууссан"-ы нэг
> хэсэг — `graphify update .`-тэй адил алхам.

### Reference-д хадгалах зүйл

- Архитектур
- Folder structure
- Module responsibility
- State lifecycle
- Data flow
- Public API contract
- Database schema
- Business rules
- UI/UX шийдвэр
- Cross-module relationships

### Reference-д хадгалахгүй зүйл

- Debug log
- Terminal output
- Stack trace
- Түр workaround
- Түр implementation detail
- TODO
- Туршилтын санаа
- Trial & Error

### Reference update хэзээ хийх вэ

**Заавал шинэчлэх** (бүтэц/логик өөрчлөгдсөн): folder structure · module responsibility · state management · data flow · database schema · public API · business rule · user workflow · architecture.

**Шинэчлэх шаардлагагүй** (зан төлөв өөрчлөгдөөгүй): CSS/styling · refactor · naming cleanup · comment · internal implementation detail.

> **6 сарын дараа ч хүчинтэй мэдлэг мөн үү?** гэж өөрөөсөө асуу. Үгүй бол reference биш — daily/plan дээр үлдээ. Эргэлзвэл reference-г бүү өөрчил.

### Reference-г хэрхэн засах вэ (AI)

✓ Холбогдох section шинэчлэх · шинэ section нэмэх · хуучирсан мэдээллийг тэмдэглэх.
✗ Document-г бүхэлд нь rewrite хийх · хүний бичсэн агуулгыг устгах · section-уудыг дур мэдэн дахин зохион байгуулах · батлагдаагүй шийдвэр нэмэх.

---

## Worklog / daily note (`E-Geree-v3-Worklog-YYYY-MM-DD.md`)

- **Нэр:** `E-Geree-v3-Worklog-YYYY-MM-DD.md` (жишээ: [[E-Geree-v3-Worklog-2026-06-25]]).
- Чөлөөт бүтэц — даалгавар, явц, шийдвэр, URL, debug тэмдэглэл, түр санаа.
- Харьцангуй огноог **үнэмлэхүй огноо** болгож бичих ("маргааш" биш "2026-06-27").

### Distill

Ажил дуусахад worklog-оос зөвхөн **урт хугацаанд хүчинтэй мэдлэгийг** холбогдох reference рүү нэгтгэнэ (worklog нь түүхэн бүртгэл хэвээр үлдэнэ).

Зарчим: ❌ "Юу болсон бэ?" → ✅ "Цаашид юу үнэн хэвээр үлдэх вэ?"

```
Daily:     StoreProvider өөрчлөгдсөн.
Reference: Product state нь StoreProvider-оор дамжин түгээгддэг болсон.
```

## Plan (`E-Geree-v3-<Name>-Plan.md`)

- **Нэр:** `E-Geree-v3-<Name>-Plan.md` — төгсгөлийн `-Plan` дагавар нь reference-ээс ялгана.
- **Frontmatter:** `title` + `tags`-д `plan` (эсвэл `egeree/plan`) + `status: proposed|in-progress|done`.
- Эхэнд: *"Энэ бол төлөвлөгөө — хэрэгжсэний дараа устгаж болно. Тогтмол reference биш."*
- Холбогдох reference рүү линк (`[[E-Geree-v3-…]]`); тогтмол хадгалах ёстой агуулга гарвал reference-д
  **нэгтгэх**, дараа нь plan-ийг устгах (хэрэгжилт + distill хоёулаа дууссан байх).
- **Documentation Impact** хэсэгтэй байх — implementation дууссаны дараа шинэчлэх reference-уудыг checkbox-оор жагсаах.

---

## Нийтлэг дүрэм

- **Хэл:** Монгол текст + Англи техник нэр/файлын зам (`src/...`, `useForm`). Кодын блок Англи.
- **Холбоос:** `[[wikilink]]` (basename-аар — бүх note нэг фолдерт flat тул basename = файлын нэр,
  жишээ: `[[E-Geree-v3-Contract-Create-Feature]]`). Markdown линк зөвхөн гадаад URL-д.
- **Кодын баримтад file path (+ боломжтой бол мөр)** иш татах — таамаг биш кодоос баталсан.
- **Жижиг үнэн нэг файлд** — нэг тэмдэглэл нэг сэдэв; давхардсан reference үүсгэхгүй,
  байгааг нь шинэчлэх.

---

## Note-ийн амьдралын мөчлөг (urgal)

```
санаа/ажил → worklog: E-Geree-v3-Worklog-YYYY-MM-DD (ноорог)
           → plan: E-Geree-v3-<Name>-Plan (хэрэгжүүлэх төлөвлөгөө, устгана)
           → distill → reference: E-Geree-v3-<Purpose> (тогтмол үнэн)
```
**Жишээ (RHF шилжилт):** 2026-06-25-ны өдрийн явц [[E-Geree-v3-Worklog-2026-06-25]]-д бичигдэв →
тогтмол state/lifecycle мэдлэг [[E-Geree-v3-Contract-Create-Feature]]-д нэгтгэгдэв → шилжилтийн
plan (E-Geree-v3-Contract-Create-RHF-Plan) хэрэгжиж 2026-06-30-нд `status: done` болов →
distill дуусаад 2026-07-07-нд энэ мөчлөгийн дүрмээр устгагдав.

---

## AI session (Claude)-д дагах дүрэм

- Кодын тухай асуултад эхлээд **graphify** (`graphify-out/` байгаа тул) — дараа нь файл унших.
- **Reference-ийг шууд том өөрчлөхгүй** — эхлээд worklog/plan-д ноорогложоод distill хийх;
  reference-д зөвхөн баталсан, тогтмол үнэн орно.
- Reference бичихдээ **кодоос баталсан file path/мөр** ашиглах; gap/таамгийг тэмдэглэх.
- **Кодын бүтэц/логик өөрчилсөн бол** холбогдох reference-г мөн адил update хийх —
  ажлын "дууссан"-ы салшгүй хэсэг.
- Шинэ `[[линк]]` нэмэхдээ **зорилтот note байгаа эсэхийг шалгах**.
- **Vault doc-ийг production кодтой хольж commit хийхгүй** (vault нь repo-гаас гадна).
- Vault-г кодоор overwrite хийхгүй, хүний бичсэн documentation-г устгахгүй — шинэ мэдлэгийг **merge** хийнэ.
- Огноог үнэмлэхүй болгож бичих.

### Ажил дууссан гэж үзэх шалгуур

- [ ] Код хэрэгжсэн
- [ ] Холбогдох reference шинэчлэгдсэн
- [ ] Worklog (daily note) шинэчлэгдсэн
- [ ] Plan obsolete болсон бол устгах боломжтой
- [ ] `graphify update .` шаардлагатай эсэхийг үнэлсэн
- [ ] Plan-ийн frontmatter status-ыг бодит төлөвт нийцүүлсэн
- [ ] Live/browser-verify хийсэн эсвэл яагаад боломжгүйг plan-д тэмдэглэсэн

> Сүүлийн 2 шалгуур — 2026-07-07-ны plan аудитын сургамж: 8 plan-ы `status:` бодит байдлаас
> хоцорсон, live/browser verification бараг бүх plan-д системтэйгээр хойшилж байсан.

---

## Загвар (skeleton-ууд)

**Reference** (`E-Geree-v3-<Purpose>.md`):
```markdown
---
title: "Гарчиг"
type: project
status: active
created: 2026-07-07
updated: 2026-07-07
tags:
  - project
  - project/e-geree-v3
---
# Гарчиг
[[E-Geree-v3-Home]] · өмнөх: [[E-Geree-v3-…]] · дараах: [[E-Geree-v3-…]]
...
```

**Plan** (`E-Geree-v3-<Name>-Plan.md`):
```markdown
---
title: "<Гарчиг>"
type: project
status: proposed
created: 2026-07-07
updated: 2026-07-07
tags:
  - project
  - plan
  - project/e-geree-v3
---
# <Гарчиг>
> [!note] Хэрэгжсэний дараа устгаж болно. Тогтмол reference биш.
[[<холбогдох reference>]]

## Documentation Impact
- [ ] [[E-Geree-v3-Architecture]]
- [ ] [[E-Geree-v3-State-Management]]
- [ ] None
```

**Worklog:** `E-Geree-v3-Worklog-YYYY-MM-DD.md` — чөлөөт бүтэц (даалгавар → явц → шийдвэр), frontmatter заавал биш.
