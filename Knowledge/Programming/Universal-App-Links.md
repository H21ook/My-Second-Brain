---
title: "Universal-App-Links"
type: knowledge
status: active
created: 2026-07-07
updated: 2026-07-07
tags:
  - knowledge
  - programming
  - mobile
  - deep-linking
  - ios
  - android
  - expo
---

# Universal Links / App Links архитектур

## Summary

Нэг каноник HTTPS URL (жишээ нь `https://yourdomain.com/products/123`) бүх платформ дээр зөв зүйл нээдэг архитектур: browser-т вэб хуудас, апп суулгасан төхөөрөмж дээр апп-ын харгалзах дэлгэц, апп байхгүй үед вэб fallback. iOS дээр **Universal Links**, Android дээр **App Links** гэж нэрлэдэг ба хоёулаа домэйн эзэмшлийг `/.well-known/` доорх verification файлаар баталгаажуулдаг.

## Problem

Custom scheme (`myapp://product/123`) хэрэглэвэл:

- Browser-т ажиллахгүй, апп суугаагүй үед юу ч нээгдэхгүй.
- SEO index-лэгдэхгүй, social/messenger апп-ууд ихэвчлэн хаадаг.
- QR, SMS, push, email гэх мэт суваг бүрд өөр link format барих шаардлага гарч логик салангид болдог.

Universal/App Links нь эдгээрийг нэг HTTPS URL-аар шийднэ: shareable, SEO-friendly, OS-ийн native дэмжлэгтэй, fallback нь үргэлж вэб.

## Core Concepts

- **Каноник URL** — resource бүрд ганц HTTPS зам (`/products/:id` маягийн). Вэб route ба апп route ижил бүтэцтэй байна.
- **Verification файлууд** — домэйн эзэмшлийг OS-д нотлох:
  - iOS: `/.well-known/apple-app-site-association`
  - Android: `/.well-known/assetlinks.json`
- **Entry points** — QR, SMS, Messenger, Email, Push notification, энгийн вэб линк: бүгд ЯГ ижил URL агуулна.
- **Fallback = вэб** — апп суугаагүй бол OS тухайн URL-ыг browser-т нээнэ; тиймээс URL бүрд ажиллах вэб хуудас байх ёстой.

## How It Works

```text
User → https://yourdomain.com/products/123
         │
   ┌─────┴──────┐
Апп суусан    Апп байхгүй
   │              │
Mobile app     Website
(detail screen) (product page)
```

1. Хэрэглэгч URL дээр дарна (эсвэл QR уншуулна).
2. OS домэйны verification файлыг урьдчилан шалгасан байдаг тул тухайн URL-ыг апп-д дамжуулах эсэхээ шийднэ.
3. Апп URL-ыг хүлээж аваад host + path-аас route/ID-г parse хийж дотоод navigation хийнэ.
4. Апп байхгүй бол ердийн browser нээгдэнэ.

### Android (App Links)

1. `/.well-known/assetlinks.json` — package name + release signing key-ийн SHA-256 fingerprint:

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.company.app",
      "sha256_cert_fingerprints": ["YOUR_SHA256"]
    }
  }
]
```

2. `AndroidManifest.xml`-д `autoVerify` бүхий intent-filter:

```xml
<intent-filter android:autoVerify="true">
  <action android:name="android.intent.action.VIEW"/>
  <category android:name="android.intent.category.DEFAULT"/>
  <category android:name="android.intent.category.BROWSABLE"/>
  <data android:scheme="https" android:host="yourdomain.com"/>
</intent-filter>
```

3. Ирсэн URL-аас route extract хийж navigate.

### iOS (Universal Links)

1. `/.well-known/apple-app-site-association` — **file extension-гүй, redirect-гүй, HTTPS-ээр** served байх шаардлагатай:

```json
{
  "applinks": {
    "details": [
      { "appID": "TEAMID.com.company.app", "paths": ["/products/*", "/orders/*"] }
    ]
  }
}
```

2. Xcode → Signing & Capabilities → Associated Domains: `applinks:yourdomain.com`.
3. Апп URL хүлээж аваад parse + navigate.

### Expo

`app.json` дотор хоёр платформын тохиргоог нэг дор:

```json
{
  "expo": {
    "scheme": "yourapp",
    "ios": { "associatedDomains": ["applinks:yourdomain.com"] },
    "android": {
      "intentFilters": [
        {
          "action": "VIEW",
          "autoVerify": true,
          "data": [{ "scheme": "https", "host": "yourdomain.com" }],
          "category": ["BROWSABLE", "DEFAULT"]
        }
      ]
    }
  }
}
```

URL унших: `Linking.getInitialURL()` (cold start), `Linking.addEventListener("url")` (running app), эсвэл Expo Router-ийн автомат route mapping.

### Апп доторх QR scanner

Апп дотроос QR уншихад Universal Link-ийг ДАХИН launch хийхгүй — апп аль хэдийн navigation-аа эзэмшиж байгаа тул зөвхөн parse хийнэ:

```text
Scan → URL string → домэйн validate → route/ID extract → дотоод navigate → API fetch → render
```

## Best Practices

- Зөвхөн HTTPS URL ашиглах; QR-д ч, SMS-д ч, push-д ч ижил каноник URL кодлох.
- QR-д хэзээ ч `product:123` эсвэл `myapp://...` кодлохгүй — зөвхөн бүрэн HTTPS URL.
- QR-д public identifier-аас өөр дотоод мэдээлэл (DB id, токен г.м.) оруулахгүй.
- Scan хийсэн URL бүрийг navigate хийхээс өмнө шалгах: HTTPS эсэх, итгэмжлэгдсэн домэйн мөн эсэх, дэмжигдсэн route мөн эсэх. `javascript:`, `ftp:`, гадаад домэйн — reject.
- Deep link бүрийн fallback нь вэб хуудас байхаар route-уудаа зохион байгуулах.
- Алдааны төлөвүүдийг тусад нь харуулах: unknown domain / unsupported route / resource not found / offline.

## Common Mistakes

- **AASA файлыг redirect-тэй эсвэл `.json` extension-тэй serve хийх** — iOS verification чимээгүй унана.
- **assetlinks.json-д debug key-ийн fingerprint үлдээх** — release build дээр App Links ажиллахгүй.
- **Апп дотроос universal link-ийг дахин нээх** — loop эсвэл browser руу үсрэх; parse хийх ёстой.
- **Вэб fallback-гүй route** — апп байхгүй хэрэглэгч 404 үзнэ.
- **Платформ бүрд өөр URL format** — нэг каноник URL-ын гол ашгийг алдана.

## Trade-offs

- **+** Нэг URL бүх суваг (QR/SMS/push/email/messenger) болон бүх платформыг дамждаг; SEO, share, fallback үнэгүй ирнэ.
- **−** Домэйн эзэмшил + verification файлын hosting шаардлагатай (backend dependency); OS-ийн verification cache-ээс болж тохиргооны алдаа оношлоход удаан.
- **−** Custom scheme-ээс ялгаатай нь OS шийдвэрийг хийдэг тул "заавал апп-д нээх" гэсэн баталгаа өгөхгүй (iOS-д хэрэглэгч browser-т нээхийг сонгож болно).

## Related Notes

- [[E-Geree-v3-EMongolia-Auth]] — гадаад auth апп-аас (e-Mongolia) буцаж ирэх redirect/deep-link хэрэглээний жишээ.

## References

- Android App Links: developer.android.com — "Verify Android App Links"
- iOS Universal Links: developer.apple.com — "Supporting associated domains"
- Expo Linking: docs.expo.dev — "Deep linking"
