# Universal App Links Architecture Guide

> **Version:** 1.0  
> **Purpose:** Define how Universal Links / App Links work and how they should be implemented across Web, Android, iOS, and Expo applications.

---

# Overview

This document describes the architecture for opening the same content from:

- Website
    
- QR Codes
    
- SMS
    
- Messenger
    
- Email
    
- Push Notifications
    
- Mobile App
    

using **one HTTPS URL**.

Instead of maintaining different link formats for different platforms, the system should expose a single canonical URL.

Example:

```text
https://yourdomain.com/products/123
```

This URL should work everywhere.

---

# Goal

One URL should support every platform.

|Platform|Result|
|---|---|
|Browser|Opens Website|
|Android App Installed|Opens Android App|
|iPhone App Installed|Opens iOS App|
|App not installed|Opens Website|
|QR Scan|Opens App or Website|
|SMS|Opens App or Website|
|Messenger|Opens App or Website|
|Email|Opens App or Website|

---

# Why Universal Links?

Avoid this:

```text
myapp://product/123
```

Problems:

- Doesn't work in browsers
    
- Doesn't work without app
    
- Cannot be indexed
    
- Often blocked by social apps
    

Instead use

```text
https://yourdomain.com/products/123
```

Benefits

- SEO friendly
    
- Shareable
    
- Works everywhere
    
- Secure
    
- Native support by Android & iOS
    

---

# URL Structure

Recommended routes

```text
/products/:productId

/orders/:orderId

/categories/:categoryId

/profile/:userId

/chat/:sessionId
```

Example

```text
https://yourdomain.com/products/123
```

---

# High Level Architecture

```text
                User

                  │
        https://yourdomain.com/products/123

                  │

        ┌─────────┴─────────┐

        │                   │

 App Installed          No App Installed

        │                   │

 Mobile App          Website Product Page

        │

 Product Detail Screen
```

---

# QR Code Architecture

QR codes should contain exactly the same URL.

Example

```text
https://yourdomain.com/products/123
```

Never encode

```text
product:123
```

or

```text
myapp://product/123
```

Always encode

```text
https://yourdomain.com/products/123
```

Benefits

- Phone Camera works
    
- App Scanner works
    
- Browser works
    
- Share works
    

---

# Website Flow

Website generates QR.

```text
Website

↓

Generate QR

↓

https://yourdomain.com/products/123
```

User scans using phone camera.

```text
Camera

↓

Universal Link

↓

App Installed?

↓

Yes → Open App

No → Open Website
```

---

# Mobile App QR Scanner Flow

Inside the mobile app

```text
User opens Scanner

↓

Camera scans QR

↓

Read String

↓

https://yourdomain.com/products/123

↓

Validate Domain

↓

Extract Product ID

↓

Navigate Product Detail
```

Important:

Do NOT launch Universal Link from inside the app.

The app already owns the navigation.

Simply parse the URL.

---

# Internal Navigation Flow

```text
QR Scanner

↓

URL

↓

Router

↓

Product Screen

↓

Fetch Product

↓

Render UI
```

---

# URL Parsing Example

Input

```text
https://yourdomain.com/products/123
```

Parse

```text
Host

yourdomain.com
```

Path

```text
/products/123
```

Extract

```text
Product ID = 123
```

Navigate

```text
ProductDetail(123)
```

---

# Error Handling

Invalid domain

```text
Unknown QR Code
```

Unknown route

```text
Unsupported QR
```

Missing Product

```text
Product Not Found
```

Offline

```text
Unable to load product.
```

---

# Security

Always verify

- HTTPS
    
- Trusted domain
    
- Supported route
    

Reject

```text
https://fake-site.com/products/123
```

Reject

```text
javascript:...
```

Reject

```text
ftp://...
```

---

# Android Implementation

Android uses **App Links**.

## Step 1

Create

```text
/.well-known/assetlinks.json
```

Example

```json
[
  {
    "relation": [
      "delegate_permission/common.handle_all_urls"
    ],
    "target": {
      "namespace": "android_app",
      "package_name": "com.company.app",
      "sha256_cert_fingerprints": [
        "YOUR_SHA256"
      ]
    }
  }
]
```

---

## Step 2

AndroidManifest.xml

```xml
<intent-filter android:autoVerify="true">

    <action android:name="android.intent.action.VIEW"/>

    <category android:name="android.intent.category.DEFAULT"/>

    <category android:name="android.intent.category.BROWSABLE"/>

    <data
        android:scheme="https"
        android:host="yourdomain.com"/>

</intent-filter>
```

---

## Step 3

Handle URL

Read incoming URL

```text
https://yourdomain.com/products/123
```

Extract route

Navigate

---

# iOS Implementation

iOS uses Universal Links.

## Step 1

Create

```text
/.well-known/apple-app-site-association
```

Example

```json
{
  "applinks": {
    "details": [
      {
        "appID": "TEAMID.com.company.app",
        "paths": [
          "/products/*",
          "/orders/*"
        ]
      }
    ]
  }
}
```

No file extension.

No redirects.

Served over HTTPS.

---

## Step 2

Enable

Associated Domains

```
applinks:yourdomain.com
```

inside Xcode Signing & Capabilities.

---

## Step 3

Receive URL

Extract

Navigate

---

# Expo Implementation

Expo supports deep linking and Universal/App Links.

## app.json

```json
{
  "expo": {
    "scheme": "yourapp",

    "ios": {
      "associatedDomains": [
        "applinks:yourdomain.com"
      ]
    },

    "android": {
      "intentFilters": [
        {
          "action": "VIEW",
          "autoVerify": true,
          "data": [
            {
              "scheme": "https",
              "host": "yourdomain.com"
            }
          ],
          "category": [
            "BROWSABLE",
            "DEFAULT"
          ]
        }
      ]
    }
  }
}
```

---

## Reading URL

Use

```ts
Linking.getInitialURL()
```

or

```ts
Linking.addEventListener("url")
```

or

Expo Router automatic routing.

---

# Recommended Folder Structure

```text
app/

    products/

        [id].tsx

components/

services/

deep-link/

    parser.ts

    routes.ts

scanner/

navigation/
```

---

# Backend Requirements

Website must expose

```text
/.well-known/apple-app-site-association
```

and

```text
/.well-known/assetlinks.json
```

Also provide APIs

```text
GET /products/:id

GET /orders/:id
```

---

# Full End-to-End Flow

```text
Admin

↓

Creates Product

↓

Website

↓

Generate QR

↓

QR

↓

https://yourdomain.com/products/123

↓

User scans

↓

OS

↓

App Installed?

↓

YES

↓

Open App

↓

Product Detail

──────────────

NO

↓

Website Product Page
```

---

# App Scanner Flow

```text
Scanner

↓

Read QR

↓

Validate URL

↓

Extract Route

↓

Extract ID

↓

Navigate

↓

Load API

↓

Display Product
```

---

# Best Practices

- Use only HTTPS URLs.
    
- Use one canonical URL for every resource.
    
- Never embed internal database information other than the public identifier in QR codes.
    
- Validate every scanned URL before navigating.
    
- Keep the web route and app route consistent.
    
- Treat the website as the fallback for every deep link.
    
- Reuse the same URLs in QR codes, emails, push notifications, SMS, Messenger, and marketing campaigns.
    

---

# Final Recommendation

Use this architecture consistently:

```text
Website
      │
      │
      ▼
https://yourdomain.com/products/:id
      │
      ├─────────────── Browser
      │
      ├─────────────── QR
      │
      ├─────────────── SMS
      │
      ├─────────────── Email
      │
      ├─────────────── Messenger
      │
      ├─────────────── Push Notification
      │
      ▼
Universal Link / App Link
      │
      ▼
Mobile App
      │
      ▼
Product Detail Screen
```

This provides a single, predictable URL architecture that scales cleanly across web, Android, iOS, Expo, and future client applications.