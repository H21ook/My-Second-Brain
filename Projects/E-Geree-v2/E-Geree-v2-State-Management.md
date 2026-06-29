---
title: "05 — Төлөв удирдлага (Context + React Query)"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v2
source_path: "D:/own/obsidian-vaults/E-Geree-v2-docs/05-State-Management.md"
---

# State Management — React Context + React Query

[[E-Geree-v2-Home]] · өмнөх: [[E-Geree-v2-Contract-Create-Feature]] · дараах: [[E-Geree-v2-Networking]]

Redux/Zustand/MobX **байхгүй**. Хоёр механизм:

## 1. React Context — UI / session / editor төлөв
21 provider-ийг `src/components/context/providers.jsx`-д угсарч,
`src/app/[locale]/layout.jsx`-д mount хийдэг.

| Context hook | Хариуцах хэсэг | Граф (edge) |
|---|---|---|
| `useToast()` | Toast мэдэгдэл (хамгийн их холбоостой) | 106 |
| `useAuth()` | Нэвтрэлт, токен, session | 60 |
| `useProfile()` | Хэрэглэгчийн профайл | 59 |
| `usePayment()` | Төлбөр, захиалга | 35 |
| `useContractState()` | Гэрээ үүсгэх wizard | 33 |
| `useDocumentLayout()` | Баримтын layout | 23 |
| `useTopLoaderRouter()` | Navigation + дээд loader | 68 |
| `useNotification()` | Мэдэгдэл | — |
| `useCompany()` / org verification | Байгууллагын профайл | — |
| `useTeams()` | Багийн удирдлага | — |
| `useFile()` / `useLabel()` / `useTable()` | Файл, шошго, хүснэгт | — |

## 2. React Query — server дата
Server Action-ийн дуудлагыг `useQuery` / `useMutation`-д ороож cache-лнэ.
Сервер дата ба клиент төлөв тусдаа — Context нь UI, React Query нь API дата.

## Анхаарах
- Leaf компонентууд props биш, шууд context руу ханддаг тул `auth-context`,
  `toast-provider`, `profile-context`-ийн өөрчлөлт өргөн нөлөөтэй.
- Шинэ provider нэмбэл `providers.jsx`-д угсарна, бусад газар DnD/Provider давхардуулахгүй.
