# Layer 2 — API Endpoints

## Base URL

```
/api/v1
```

All endpoints return JSON. Authentication via `Authorization: Bearer <jwt>` header.

---

## Auth — `/api/v1/auth`

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| POST | `/auth/register` | إنشاء حساب جديد | Public |
| POST | `/auth/login` | تسجيل الدخول → JWT | Public |
| POST | `/auth/refresh` | تجديد access token | Refresh token |
| POST | `/auth/logout` | إبطال refresh token | User |
| POST | `/auth/verify-email` | تأكيد البريد | Public (token) |
| POST | `/auth/resend-verification` | إعادة إرسال بريد التأكيد | Public |
| POST | `/auth/forgot-password` | طلب إعادة تعيين كلمة المرور | Public |
| POST | `/auth/reset-password` | تعيين كلمة مرور جديدة | Public (token) |

### `POST /auth/register`

```typescript
// Request
{
  email: string,        // "user@example.com"
  password: string,     // min 8 chars
  name?: string
}

// Response 201
{
  message: "verification email sent",
  userId: "uuid"
}
```

### `POST /auth/login`

```typescript
// Request
{
  email: string,
  password: string
}

// Response 200
{
  accessToken: string,    // expires in 15m
  refreshToken: string,   // expires in 7d
  user: {
    id: string,
    email: string,
    name: string,
    role: "user" | "admin",
    emailVerified: boolean
  }
}
```

---

## Users — `/api/v1/users`

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | `/users/me` | الملف الشخصي | User |
| PATCH | `/users/me` | تحديث الملف الشخصي | User |
| PATCH | `/users/me/password` | تغيير كلمة المرور | User |
| DELETE | `/users/me` | حذف الحساب | User |
| GET | `/users/me/plan` | الخطة الحالية + الحدود | User |

### `GET /users/me`

```typescript
// Response 200
{
  id: string,
  email: string,
  name: string,
  role: string,
  status: string,
  plan: {
    name: string,
    maxInstances: number,
    maxChannels: number,
    maxMessagesMonth: number
  },
  usage: {
    instancesCount: number,
    channelsCount: number,
    messagesThisMonth: number
  },
  createdAt: string
}
```

---

## Instances — `/api/v1/instances`

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | `/instances` | قائمة instances المستخدم | User |
| POST | `/instances` | إنشاء instance جديد | User |
| GET | `/instances/:id` | تفاصيل instance | User (owner) |
| PATCH | `/instances/:id` | تحديث إعدادات | User (owner) |
| DELETE | `/instances/:id` | حذف instance | User (owner) |
| POST | `/instances/:id/start` | تشغيل | User (owner) |
| POST | `/instances/:id/stop` | إيقاف | User (owner) |
| POST | `/instances/:id/restart` | إعادة تشغيل | User (owner) |
| GET | `/instances/:id/status` | حالة مباشرة (من الـ node) | User (owner) |
| GET | `/instances/:id/logs` | آخر السجلات | User (owner) |

### `POST /instances`

```typescript
// Request
{
  name: string,           // "مساعد متجري"
  region?: string,        // "nyc1" default
}

// Response 202 (Accepted — provisioning started)
{
  id: string,
  name: string,
  status: "creating",
  message: "instance is being provisioned"
}
```

### `GET /instances/:id/status`

```typescript
// Response 200 — real-time status from openclaw gateway
{
  id: string,
  name: string,
  status: "running" | "stopped" | "error",
  gateway: {
    version: string,
    uptime: number,          // seconds
    port: number
  },
  channels: [
    {
      type: "whatsapp",
      status: "connected",
      accountId: "966xxxxxxx"
    },
    {
      type: "telegram",
      status: "disconnected"
    }
  ],
  resources: {
    cpuPercent: number,
    memoryUsedMb: number,
    diskUsedPercent: number
  }
}
```

---

## Channels — `/api/v1/instances/:id/channels`

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | `/instances/:id/channels` | قائمة القنوات | User (owner) |
| POST | `/instances/:id/channels` | ربط قناة جديدة | User (owner) |
| GET | `/instances/:id/channels/:channelId` | تفاصيل قناة | User (owner) |
| DELETE | `/instances/:id/channels/:channelId` | فصل قناة | User (owner) |
| POST | `/instances/:id/channels/:channelId/reconnect` | إعادة اتصال | User (owner) |
| GET | `/instances/:id/channels/whatsapp/qr` | QR code للواتساب | User (owner) |

### `POST /instances/:id/channels`

```typescript
// Request
{
  type: "whatsapp" | "telegram" | "discord" | "slack" | "signal",
  config?: {
    // Telegram
    botToken?: string,
    // Discord
    botToken?: string,
    guildId?: string,
    // Slack
    appToken?: string,
    botToken?: string
  }
}

// Response 202
{
  channelId: string,
  type: string,
  status: "connecting",
  // For WhatsApp — qr will arrive via WebSocket
  message: "channel linking initiated"
}
```

### `GET /instances/:id/channels/whatsapp/qr`

```typescript
// Response 200
{
  qrCode: string,     // base64-encoded QR image
  expiresIn: number    // seconds until QR expires
}

// Response 409 — already connected
{
  error: "whatsapp already connected"
}
```

---

## Billing — `/api/v1/billing`

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | `/billing/plans` | الخطط المتاحة | Public |
| GET | `/billing/subscription` | اشتراك المستخدم | User |
| POST | `/billing/subscribe` | بدء اشتراك (Stripe Checkout) | User |
| POST | `/billing/change-plan` | تغيير الخطة | User |
| POST | `/billing/cancel` | إلغاء الاشتراك | User |
| GET | `/billing/invoices` | قائمة الفواتير | User |
| GET | `/billing/invoices/:id/pdf` | تحميل فاتورة | User |
| POST | `/billing/webhook` | Stripe webhook | Stripe signature |

### `POST /billing/subscribe`

```typescript
// Request
{
  planId: string,
  billingCycle: "monthly" | "yearly"
}

// Response 200
{
  checkoutUrl: string,     // Stripe Checkout Session URL
  sessionId: string
}
```

---

## Usage — `/api/v1/usage`

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | `/usage` | ملخص الاستخدام الشهري | User |
| GET | `/usage/history` | تاريخ الاستخدام (آخر 12 شهر) | User |
| GET | `/usage/instances/:id` | استخدام instance محدد | User (owner) |

### `GET /usage`

```typescript
// Response 200
{
  currentPeriod: {
    start: string,
    end: string
  },
  totals: {
    messagesSent: number,
    messagesReceived: number,
    tokensUsed: number
  },
  limits: {
    maxMessages: number,
    usagePercent: number     // 0-100
  },
  perInstance: [
    {
      instanceId: string,
      instanceName: string,
      messagesSent: number,
      messagesReceived: number
    }
  ]
}
```

---

## Admin — `/api/v1/admin`

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| **Users** | | | |
| GET | `/admin/users` | قائمة كل المستخدمين | Admin |
| GET | `/admin/users/:id` | تفاصيل مستخدم | Admin |
| PATCH | `/admin/users/:id` | تعديل مستخدم (role, plan) | Admin |
| POST | `/admin/users/:id/suspend` | تعليق حساب | Admin |
| POST | `/admin/users/:id/unsuspend` | إلغاء التعليق | Admin |
| **Instances** | | | |
| GET | `/admin/instances` | كل الـ instances | Admin |
| GET | `/admin/instances/:id` | تفاصيل instance (مع owner) | Admin |
| POST | `/admin/instances/:id/force-stop` | إيقاف إجباري | Admin |
| **Nodes** | | | |
| GET | `/admin/nodes` | كل السيرفرات | Admin |
| POST | `/admin/nodes` | إضافة node جديد | Admin |
| GET | `/admin/nodes/:id` | تفاصيل node | Admin |
| PATCH | `/admin/nodes/:id` | تعديل node | Admin |
| DELETE | `/admin/nodes/:id` | حذف node | Admin |
| POST | `/admin/nodes/:id/health-check` | فحص صحة node | Admin |
| **Revenue** | | | |
| GET | `/admin/revenue/summary` | ملخص الإيرادات | Admin |
| GET | `/admin/revenue/mrr` | MRR chart data | Admin |
| **System** | | | |
| GET | `/admin/system/health` | صحة كل الخدمات | Admin |
| GET | `/admin/audit-logs` | سجل المراجعة | Admin |
| **Announcements** | | | |
| GET | `/admin/announcements` | كل الإعلانات | Admin |
| POST | `/admin/announcements` | إنشاء إعلان | Admin |
| PATCH | `/admin/announcements/:id` | تعديل إعلان | Admin |
| DELETE | `/admin/announcements/:id` | حذف إعلان | Admin |

### `GET /admin/users?page=1&limit=20&search=ahmed&status=active`

```typescript
// Response 200
{
  users: [
    {
      id: string,
      email: string,
      name: string,
      role: string,
      status: string,
      plan: string,
      instancesCount: number,
      messagesThisMonth: number,
      createdAt: string,
      lastLoginAt: string
    }
  ],
  pagination: {
    page: number,
    limit: number,
    total: number,
    totalPages: number
  }
}
```

### `GET /admin/system/health`

```typescript
// Response 200
{
  status: "healthy" | "degraded" | "down",
  services: {
    api: { status: "up", responseTime: 12 },
    database: { status: "up", responseTime: 3 },
    redis: { status: "up", responseTime: 1 },
    nodes: {
      total: 10,
      healthy: 9,
      unhealthy: 1,
      details: [
        { id: string, ip: string, status: "running", lastCheck: string }
      ]
    }
  },
  uptime: number,
  version: string
}
```

---

## Health — `/api/v1/health`

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | `/health` | فحص سريع | Public |
| GET | `/health/ready` | فحص شامل | Public |

### `GET /health`

```typescript
// Response 200
{ status: "ok", timestamp: string }
```

---

## WebSocket Events

### الاتصال

```
ws://api.openclaw.ai/ws?token=<jwt>
```

### الأحداث من Server → Client

| Event | Payload | الوصف |
|-------|---------|-------|
| `instance:status` | `{ instanceId, status, details }` | تغيّر حالة instance |
| `instance:provision:progress` | `{ instanceId, step, total, message }` | تقدّم الإنشاء |
| `channel:status` | `{ instanceId, channelId, type, status }` | تغيّر حالة قناة |
| `channel:qr` | `{ instanceId, qrBase64, expiresIn }` | QR code جاهز |
| `usage:warning` | `{ instanceId, usagePercent, limit }` | تحذير استخدام |
| `system:alert` | `{ type, severity, message }` | تنبيه نظام (admin) |
| `announcement:new` | `{ id, title, type }` | إعلان جديد |

### غرف (Rooms)

```typescript
// كل مستخدم ينضم تلقائياً لغرفته
user:<userId>         // أحداث خاصة بالمستخدم

// كل instance لها غرفة
instance:<instanceId> // أحداث خاصة بالـ instance

// Admin
admin                 // تنبيهات النظام
```

---

## أكواد الخطأ

```typescript
// Standard error response
{
  error: {
    code: string,         // "VALIDATION_ERROR" | "NOT_FOUND" | ...
    message: string,      // رسالة واضحة
    details?: object      // تفاصيل إضافية (validation errors)
  }
}
```

| HTTP Status | Code | الوصف |
|-------------|------|-------|
| 400 | `VALIDATION_ERROR` | بيانات الطلب غير صحيحة |
| 401 | `UNAUTHORIZED` | غير مسجّل الدخول |
| 403 | `FORBIDDEN` | لا صلاحية |
| 404 | `NOT_FOUND` | المورد غير موجود |
| 409 | `CONFLICT` | تعارض (مثل email مكرر) |
| 422 | `PLAN_LIMIT_REACHED` | تجاوز حد الخطة |
| 429 | `RATE_LIMITED` | طلبات كثيرة |
| 500 | `INTERNAL_ERROR` | خطأ داخلي |
| 502 | `NODE_UNREACHABLE` | السيرفر غير متاح |
| 503 | `SERVICE_UNAVAILABLE` | الخدمة معطّلة |

---

## Pagination & Filtering (عام)

كل endpoint يرجع قوائم يدعم:

```
?page=1           ← رقم الصفحة (default: 1)
?limit=20         ← عدد العناصر (default: 20, max: 100)
?sort=created_at  ← حقل الترتيب
?order=desc       ← asc | desc
?search=keyword   ← بحث نصي
?status=active    ← فلترة حسب الحالة
```

---

## Rate Limits

| Scope | Limit |
|-------|-------|
| Auth endpoints | 10 req/min per IP |
| General API | 100 req/min per user |
| Admin API | 200 req/min per user |
| WebSocket events | 50 events/min per user |
| Webhook | 100 req/min per source |
