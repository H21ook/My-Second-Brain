---
title: "E-Geree-v2"
type: project
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - project
  - imported
  - project/e-geree-v2
source_path: "D:/own/obsidian-vaults/E-Geree-v2-docs/_HOME.md"
---

# E-Geree-v2 — Төслийн баримт

Next.js 14 (App Router) дээр суурилсан цахим гэрээ (e-contract) платформ. Энэ vault нь
төслийн **тойм, архитектур, гол модулиудын** гар аргаар бичсэн зураглал.

## 📚 Агуулга
1. [[E-Geree-v2-Overview]] — Төслийн тойм, технологийн стек, хавтасны бүтэц
2. [[E-Geree-v2-Architecture]] — Давхаргат архитектур, дата урсгал, гол зарчмууд
3. [[E-Geree-v2-Routing]] — Next.js App Router бүтэц
4. [[E-Geree-v2-Contract-Create-Feature]] — Гэрээ үүсгэх wizard (гол feature)
5. [[E-Geree-v2-State-Management]] — React Context + React Query
6. [[E-Geree-v2-Networking]] — Action → Service → HTTP давхарга
7. [[E-Geree-v2-PDF-Editor]] — PDF гэрээ засварлагч ба талбар (field) editor

## 🎯 Богино тайлбар
E-Geree нь **гэрээг дижитал хэлбэрээр үүсгэх, талбар (гарын үсэг, огноо г.м.)
байршуулах, оролцогчдод илгээж, гарын үсэг цуглуулах** платформ. Цэвэр frontend —
дотроо database/ORM байхгүй, бүх дата гадаад микросервисээс HTTP/WebSocket-ээр ирнэ.

> Энэ нь graphify-ийн автомат граф биш. Бүрэн node-граф нь `graphify-out/graph.html`,
> `graphify-out/GRAPH_REPORT.md`-д тусдаа байгаа.
