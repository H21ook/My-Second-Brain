---
title: "E-Geree-v3"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/_HOME.md"
---

# E-Geree-v3 — Төслийн баримт

Next.js 16 + React 19 дээр суурилсан цахим гэрээ (e-contract) платформ. Энэ vault нь
төслийн **тойм, архитектур, гол модулиудын** гар аргаар бичсэн зураглал.

## 📚 Агуулга
0. [[E-Geree-v3-Note-Keeping-Guide]] — Тэмдэглэл хөтлөх заавар (vault конвенц)
1. [[E-Geree-v3-Overview]] — Төслийн тойм, технологийн стек, хавтасны бүтэц
2. [[E-Geree-v3-Architecture]] — Давхаргат архитектур, дата урсгал, гол зарчмууд
3. [[E-Geree-v3-Routing]] — Next.js App Router бүтэц
4. [[E-Geree-v3-Contract-Create-Feature]] — Гэрээ үүсгэх wizard (гол feature)
5. [[E-Geree-v3-State-Management]] — Redux Toolkit store, slice-ууд
6. [[E-Geree-v3-Networking-BFF]] — BFF давхарга, fetcher-ууд, proxy
7. [[E-Geree-v3-PDF-Viewer]] — PDF үзэгч ба талбар (field) editor

## 🎯 Богино тайлбар
E-Geree нь **гэрээг дижитал хэлбэрээр үүсгэх, талбар (гарын үсэг, огноо г.м.)
байршуулах, оролцогчдод илгээж, гарын үсэг цуглуулах** платформ. Гол ажлын урсгал нь
олон алхамт wizard (Participant → Content → Fields → Submit) бөгөөд PDF дээр шууд
талбар тавьдаг.

> Энэ нь graphify-ийн автомат граф биш. Бүрэн node-граф нь `graphify-out/graph.html`,
> `graphify-out/GRAPH_REPORT.md`-д тусдаа байгаа.
