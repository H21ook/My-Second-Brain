---
title: "00 — Тэмдэглэл хөтлөх заавар"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
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
- **Daily Note** → Юу болсон бэ?
- **Reference** → Цаашид юу үнэн хэвээр үлдэх вэ?

AI нь үргэлж эдгээрийг ялгаж ойлгоно.

---

## Note-ийн 3 үндсэн төрөл

| Төрөл                   | Байрлал ба нэр                                                          | Frontmatter tag           | Хэзээ ашиглах                                  | Амьдралын мөчлөг                                   |
| ----------------------- | ----------------------------------------------------------------------- | ------------------------- | ---------------------------------------------- | -------------------------------------------------- |
| **Дугаартай reference** | Root: `NN-Kebab-Title.md` (`01-Overview`, `04-Contract-Create-Feature`) | `egeree/<topic>`          | Тогтмол үнэн — архитектур, feature, дэд систем | Удаан амьдарна; шинэчилж байна, устгахгүй          |
| **Daily note**          | `daily-notes/YYYY-MM-DD.md`                                             | (заавал биш)              | Өдрийн ажил, явц, түр санаа, debug             | Богино настай — дуусахад reference рүү **distill** |
| **Plan (устгаж болох)** | `plans/<Name>.md`                                                       | `egeree/plan` + `status:` | Хэрэгжүүлэх төлөвлөгөө/proposal                | Хэрэгжсэний дараа **устгана**                      |

> `some-prompts/` / `awesome-prompts/` (prompt template) одоо байгаа — энэ заавар түүнийг хөнддөггүй, Note системийн нэг хэсэг биш.

---

## Дугаартай reference (01–NN)

- **Нэр:** `NN-Kebab-Title.md`. Дугаар нь жагсаалтын дараалал — сэдвийн логик урсгал
  (`01-Overview` → `02-Architecture` → …).
- **Frontmatter:** `title` (Монгол, дугаартай) + `tags: [egeree/<topic>]`.
- **Breadcrumb (эхний мөрөнд):** `[[E-Geree-v3-Home]] · өмнөх: [[NN-…]] · дараах: [[NN-…]]`,
  шаардвал `· холбоотой: [[…]]`.
- **Шинэ reference нэмбэл** `_HOME`-д бүртгэх; өмнөх/дараах breadcrumb-ийг хөршүүдэд нь холбох.
- **Тогтмол үнэн байх** — өдрийн явц/ноорог энд орохгүй (тэр нь daily/plans).

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

## Daily note (`daily-notes/`)

- **Нэр:** `YYYY-MM-DD.md`.
- Чөлөөт бүтэц — даалгавар, явц, шийдвэр, URL, debug тэмдэглэл, түр санаа.
- Харьцангуй огноог **үнэмлэхүй огноо** болгож бичих ("маргааш" биш "2026-06-27").

### Distill

Ажил дуусахад daily note-оос зөвхөн **урт хугацаанд хүчинтэй мэдлэгийг** холбогдох reference рүү нэгтгэнэ (daily нь түүхэн бүртгэл хэвээр үлдэнэ).

Зарчим: ❌ "Юу болсон бэ?" → ✅ "Цаашид юу үнэн хэвээр үлдэх вэ?"

```
Daily:     StoreProvider өөрчлөгдсөн.
Reference: Product state нь StoreProvider-оор дамжин түгээгддэг болсон.
```

## Plan (`plans/`)

- **Нэр:** `<Descriptive-Name>.md` (дугааргүй — reference биш).
- **Frontmatter:** `title` + `tags: [egeree/plan]` + `status: proposed|in-progress|done`.
- Эхэнд: *"Энэ бол төлөвлөгөө — хэрэгжсэний дараа устгаж болно. Тогтмол reference биш."*
- Холбогдох reference рүү линк (`[[04-…]]`); тогтмол хадгалах ёстой агуулга гарвал reference-д
  **нэгтгэх**, дараа нь plan-ийг устгах.
- **Documentation Impact** хэсэгтэй байх — implementation дууссаны дараа шинэчлэх reference-уудыг checkbox-оор жагсаах.

---

## Нийтлэг дүрэм

- **Хэл:** Монгол текст + Англи техник нэр/файлын зам (`src/...`, `useForm`). Кодын блок Англи.
- **Холбоос:** `[[wikilink]]` (basename-аар, фолдер хамаарахгүй — `[[E-Geree-v3-RHF-Migration-Plan]]`
  нь `plans/`-д байсан ч ажиллана). Markdown линк зөвхөн гадаад URL-д.
- **Кодын баримтад file path (+ боломжтой бол мөр)** иш татах — таамаг биш кодоос баталсан.
- **Жижиг үнэн нэг файлд** — нэг тэмдэглэл нэг сэдэв; давхардсан reference үүсгэхгүй,
  байгааг нь шинэчлэх.

---

## Note-ийн амьдралын мөчлөг (urgal)

```
санаа/ажил → daily-note (ноорог)
           → plans/ (хэрэгжүүлэх төлөвлөгөө, устгана)
           → distill → дугаартай reference (тогтмол үнэн)
```
**Жишээ (2026-06-25):** өдрийн явц `daily-notes/2026-06-25.md`-д → тогтмол state/lifecycle
мэдлэг `04-Contract-Create-Feature`-д нэгтгэгдэв → RHF шилжилт нь устгаж болох
`plans/RHF-Migration-Plan.md` боллоо.

---

## AI session (Claude)-д дагах дүрэм

- Кодын тухай асуултад эхлээд **graphify** (`graphify-out/` байгаа тул) — дараа нь файл унших.
- **Дугаартай reference-ийг шууд том өөрчлөхгүй** — эхлээд daily/plans-д ноорогложоод distill хийх;
  reference-д зөвхөн баталсан, тогтмол үнэн орно.
- Reference бичихдээ **кодоос баталсан file path/мөр** ашиглах; gap/таамгийг тэмдэглэх.
- **Кодын бүтэц/логик өөрчилсөн бол** холбогдох дугаартай reference-г мөн адил update хийх —
  ажлын "дууссан"-ы салшгүй хэсэг.
- Шинэ `[[линк]]` нэмэхдээ **зорилтот note байгаа эсэхийг шалгах**.
- **Vault doc-ийг production кодтой хольж commit хийхгүй** (vault нь repo-гаас гадна).
- Vault-г кодоор overwrite хийхгүй, хүний бичсэн documentation-г устгахгүй — шинэ мэдлэгийг **merge** хийнэ.
- Огноог үнэмлэхүй болгож бичих.

### Ажил дууссан гэж үзэх шалгуур

- [ ] Код хэрэгжсэн
- [ ] Холбогдох reference шинэчлэгдсэн
- [ ] Daily note шинэчлэгдсэн
- [ ] Plan obsolete болсон бол устгах боломжтой
- [ ] `graphify update .` шаардлагатай эсэхийг үнэлсэн

---

## Загвар (skeleton-ууд)

**Дугаартай reference:**
```markdown
---
title: NN — Гарчиг
tags: [egeree/<topic>]
---
# Гарчиг
[[E-Geree-v3-Home]] · өмнөх: [[NN-…]] · дараах: [[NN-…]]
...
```

**Plan:**
```markdown
---
title: <Гарчиг>
tags: [egeree/plan]
status: proposed
---
# <Гарчиг>
> [!note] Хэрэгжсэний дараа устгаж болно. Тогтмол reference биш.
[[<холбогдох reference>]]

## Documentation Impact
- [ ] 02-Architecture
- [ ] 05-State-Management
- [ ] None
```

**Daily note:** `daily-notes/2026-06-27.md` — чөлөөт бүтэц (даалгавар → явц → шийдвэр).
