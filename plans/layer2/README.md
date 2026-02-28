# Layer 2 — Core API (الطبقة الوسيطة)

## ما هو هذا المجلد؟

خطة تفصيلية لبناء **Layer 2** — الطبقة الوسيطة (Core API) التي تربط:
- **Layer 1** (لوحة تحكم العميل + لوحة الإدارة) ← REST API
- **السيرفرات** (DigitalOcean Droplets) ← SSH/API control
- **openclaw** (الـ gateway الذي يشغّل القنوات) ← CLI/process management

---

## هيكل الملفات

```
plans/layer2/
├── README.md                          ← أنت هنا
│
├── architecture/
│   ├── overview.md                    ← البنية العامة + تدفق البيانات
│   ├── tech-stack.md                  ← التقنيات + بنية المجلدات
│   ├── database-schema.md             ← جداول PostgreSQL (Drizzle)
│   └── endpoints.md                   ← كل الـ API endpoints
│
├── sprints/
│   ├── sprint-0-setup.md              ← أسبوع | إعداد المشروع + DB + Auth
│   ├── sprint-1-users-instances.md    ← أسبوعان | المستخدمين + الـ Instances
│   ├── sprint-2-channels-control.md   ← أسبوعان | القنوات + التحكم عن بُعد
│   ├── sprint-3-billing-usage.md      ← أسبوعان | الفوترة + الاستخدام
│   └── sprint-4-admin-monitoring.md   ← أسبوعان | لوحة الإدارة + المراقبة
│
└── tasks/
    └── backlog.md                     ← P0→P3 مرتّب + قرارات
```

---

## الخلاصة التنفيذية

### الدور الرئيسي

```
┌──────────────┐     REST/WS      ┌──────────────┐    SSH/Control    ┌──────────────┐
│   Layer 1    │ ◄──────────────► │   Layer 2    │ ◄──────────────► │  VPS Nodes   │
│  Dashboard   │                  │  Core API    │                  │  (openclaw)  │
│  Admin Panel │                  │              │                  │              │
└──────────────┘                  │  PostgreSQL  │                  │  gateway     │
                                  │  Job Queue   │                  │  channels    │
                                  │  WebSocket   │                  │  sessions    │
                                  └──────────────┘                  └──────────────┘
```

Layer 2 هو **الدماغ المركزي**:
1. يستقبل طلبات من الداشبورد (Layer 1)
2. يتحكم بالسيرفرات عن بُعد (إنشاء/إيقاف/إعادة تشغيل instances)
3. يُشغّل openclaw على السيرفرات ويراقب حالتها
4. يدير المستخدمين والاشتراكات والفوترة
5. يرسل أحداث real-time للداشبورد (WebSocket)

---

### المدة الإجمالية: ~9 أسابيع

| Sprint | المدة | الهدف | النقاط |
|--------|-------|-------|--------|
| Sprint 0 | أسبوع | Setup + DB + Auth + Health | 18 |
| Sprint 1 | أسبوعان | Users + Instances + Node Control | 35 |
| Sprint 2 | أسبوعان | Channels + Gateway Control | 27 |
| Sprint 3 | أسبوعان | Billing + Usage Tracking | 25 |
| Sprint 4 | أسبوعان | Admin API + Monitoring + Audit | 30 |
| **الإجمالي** | **~9 أسابيع** | | **135 نقطة** |

---

### التقنيات المختارة

```
Hono                        ← API Framework (خفيف، سريع، TypeScript-first)
TypeScript                  ← اللغة
PostgreSQL 16               ← قاعدة البيانات ✅
Drizzle ORM                 ← Database ORM ✅
BullMQ + Redis              ← Job Queue (provisioning, billing)
Socket.io                   ← Real-time events للداشبورد
node-ssh                    ← SSH control للسيرفرات
Zod                         ← Request/Response validation
JWT (jose)                  ← Authentication tokens
Vitest                      ← Testing
DigitalOcean                ← Hosting ✅ (45.55.253.17)
```

---

### العلاقة مع Layer 1

| Layer 1 يحتاج (Sprint) | Layer 2 يوفّر (Sprint) |
|------------------------|----------------------|
| Auth endpoints (S0) | User registration + JWT (S0) |
| Instance CRUD (S2) | Instance management + SSH (S1) |
| Channel linking (S3) | Channel control via openclaw CLI (S2) |
| Billing/Usage (S3) | Stripe integration + usage tracking (S3) |
| Admin endpoints (S4) | Admin API + audit logs (S4) |

**التوصية:** Layer 2 Sprint 0-1 يجب أن ينتهي **قبل** Layer 1 Sprint 2.

---

## القرارات المُحسومة ✅

| القرار | الاختيار | التاريخ |
|--------|---------|---------|
| قاعدة البيانات | PostgreSQL 16 | 2026-02-28 |
| الـ ORM | Drizzle ORM | 2026-02-28 |
| الاستضافة | DigitalOcean | 2026-02-28 |
| IP الخادم (Dev/Staging) | `45.55.253.17` | 2026-02-28 |
| API Framework | Hono | 2026-02-28 |

## القرارات المعلّقة

| السؤال | الخيارات | يؤثر على |
|--------|---------|---------|
| هل كل مستخدم يحصل على VPS مستقل أو مشترك؟ | dedicated / shared / hybrid | S1 |
| كيف يتم provisioning السيرفرات؟ | DigitalOcean API / pre-provisioned pool | S1 |
| حدود الاستخدام لكل خطة؟ | TBD | S3 |
| هل نحتاج WebSocket أو SSE؟ | WebSocket / SSE | S1 |
