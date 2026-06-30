# E-Mongolia Authentication Flow

> Mobile болон Web дээр найдвартай ажиллах e-Mongolia (3дагч систем) нэвтрэлтийн архитектур.

---

# Асуудал

Одоогийн архитектур дараах байдлаар ажилладаг.

```text
Frontend
    │
    │ POST /start
    ▼
Backend
    │
    │ requestId + redirectUrl
    ▼
Frontend (Tab A)
    │
    ├── requestId хадгална
    ├── Socket сонсоно
    └── redirectUrl-г шинэ Tab (Tab B) дээр нээнэ
                 │
                 ▼
          e-Mongolia
                 │
                 ▼
             Backend Callback
                 │
                 ▼
          Socket Event
                 │
                 ▼
          Frontend (Tab A)
```

Desktop browser дээр энэ ажилладаг.

Харин Mobile browser дээр:

- Background tab pause болдог.
- WebSocket disconnect болох боломжтой.
- Timer болон Event-үүд ажиллахгүй байж болно.
- Зарим browser шинэ tab нээхгүй.
- Зарим browser өмнөх tab-г унтуулдаг.

Иймээс Socket дээр үндэслэсэн архитектур mobile дээр найдвартай биш.

---

# Зөв Architecture

Socket нь үндсэн механизм биш.

Socket бол зөвхөн UX сайжруулах нэмэлт боломж.

Үндсэн authentication flow нь Request Status дээр суурилсан байх ёстой.

---

# High Level Flow

```text
Frontend
    │
    ▼
Backend
    │
    ▼
e-Mongolia
    │
    ▼
Backend Callback
    │
    ├── Redis
    ├── Database
    ▼
Frontend Callback
```

---

# Sequence Diagram

```mermaid
sequenceDiagram

participant FE as Frontend
participant BE as Backend
participant EM as e-Mongolia
participant Redis
participant DB

FE->>BE: POST /api/auth/emongolia/start

BE->>Redis: SET request=PENDING
BE->>DB: INSERT auth_request

BE-->>FE:
requestId
redirectUrl

FE->>FE:
sessionStorage.set(requestId)

FE->>EM:
Redirect

EM->>BE:
POST callback

BE->>Redis:
SET SUCCESS

BE->>DB:
UPDATE SUCCESS

BE-->>FE:
302 Redirect
/auth/callback?requestId=...

FE->>BE:
GET /status

BE->>Redis:
GET request

alt Not Found
    BE->>DB:
    SELECT request
end

BE-->>FE:
SUCCESS / FAILED / PENDING
```

---

# API Design

## 1. Start Authentication

```
POST /api/auth/emongolia/start
```

Response

```json
{
    "requestId": "...",
    "redirectUrl": "..."
}
```

Backend

- requestId үүсгэнэ
- Redis дээр PENDING хадгална
- DB дээр request бүртгэнэ
- redirectUrl буцаана

---

## 2. Callback

```
POST /api/auth/emongolia/callback
```

e-Mongolia callback.

Backend

- Signature шалгана
- Token шалгана
- User мэдээлэл авна
- Status шинэчилнэ

Redis

```
SUCCESS
```

эсвэл

```
FAILED
```

---

## 3. Status API

```
GET /api/auth/emongolia/status?requestId=...
```

Response

```json
{
    "status":"SUCCESS",
    "user":{},
    "token":"..."
}
```

эсвэл

```json
{
    "status":"FAILED"
}
```

эсвэл

```json
{
    "status":"PENDING"
}
```

---

# Redis

Redis нь хурдан status шалгах зориулалттай.

Жишээ

```
auth:request:xxxx

Status=PENDING
TTL=10min
```

Callback ирэхэд

```
Status=SUCCESS
```

эсвэл

```
Status=FAILED
```

---

# Database

Database нь audit болон persistence зориулалттай.

Жишээ

```
auth_requests

id
requestId
status
createdAt
updatedAt
userId
```

---

# Frontend

## Step 1

```
POST /start
```

↓

```
requestId
redirectUrl
```

↓

```
sessionStorage
```

↓

```
Redirect
```

---

## Callback Page

```
/auth/callback
```

нээгдэхэд

```
GET /status
```

дуудна.

---

# Status

## SUCCESS

```
Dashboard
```

руу оруулна.

---

## FAILED

Алдаа харуулна.

---

## PENDING

2 секунд тутам polling.

---

# Socket ашиглах бол

Socket нь optional.

```
Backend

SUCCESS

↓

Socket Event

↓

Frontend
```

Socket ирээгүй бол

```
Polling
```

үргэлжилнэ.

---

# Хэрэв New Tab ашиглаж байгаа бол

Tab A

```
Манай систем
```

Tab B

```
e-Mongolia
```

Tab B хаагдаад хэрэглэгч Tab A руу буцаж ирэх үед

Frontend дараах event-үүдийг ашиглаж болно.

```
window.focus
```

болон

```
document.visibilitychange
```

Жишээ

```ts
window.addEventListener("focus", checkStatus)

document.addEventListener("visibilitychange", () => {
    if (document.visibilityState === "visible") {
        checkStatus()
    }
})
```

checkStatus()

↓

```
GET /status
```

↓

SUCCESS

↓

Login

---

# Mobile Recommendation

❌ Socket only

```
Tab A

Socket waiting

↓

Tab B

Authentication

↓

Socket event
```

Mobile дээр найдвартай биш.

---

## Recommended

```
Frontend

↓

POST /start

↓

Backend

↓

requestId

↓

Redirect

↓

e-Mongolia

↓

Backend Callback

↓

Redis + Database

↓

Frontend Callback

↓

GET /status

↓

SUCCESS

↓

Dashboard
```

---

# Architecture Summary

| Component | Responsibility |
|------------|----------------|
| Frontend | Authentication эхлүүлэх, requestId хадгалах, status шалгах |
| Backend | Request үүсгэх, callback боловсруулах |
| e-Mongolia | Authentication хийх |
| Redis | Түр хугацааны authentication state хадгалах |
| Database | Audit болон authentication history хадгалах |
| Socket | Optional UX improvement |
| Status API | Үндсэн synchronization механизм |

---

# Final Recommendation

✅ requestId ашиглах

✅ Redis дээр authentication state хадгалах

✅ Database дээр audit хадгалах

✅ Callback API ашиглах

✅ Status API ашиглах

✅ Mobile дээр same-tab redirect хийх

✅ Socket-ийг зөвхөн нэмэлт notification болгон ашиглах

❌ Authentication flow-г зөвхөн WebSocket дээр суурилуулахгүй.