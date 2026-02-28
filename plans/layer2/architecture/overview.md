# Layer 2 — Architecture Overview

## الدور

Layer 2 هو **Core API** — الطبقة الوسيطة التي:
- تستقبل طلبات HTTP من الداشبورد (Layer 1)
- تتحكم بسيرفرات openclaw عن بُعد عبر SSH
- تدير قاعدة البيانات (المستخدمين، الاشتراكات، الاستخدام)
- تُشغّل مهام الخلفية (provisioning، billing، monitoring)
- ترسل أحداث real-time للداشبورد

---

## التدفق العام

```
                            ┌─────────────────────────────────────────────┐
                            │              Layer 2 — Core API             │
                            │                                             │
  ┌──────────┐   REST/WS    │  ┌───────────┐  ┌───────────┐  ┌────────┐  │
  │ Layer 1  │ ◄──────────► │  │   Hono    │  │  Workers  │  │ Socket │  │
  │Dashboard │              │  │  Router   │  │ (BullMQ)  │  │  .io   │  │
  │  Admin   │              │  └─────┬─────┘  └─────┬─────┘  └───┬────┘  │
  └──────────┘              │        │              │             │       │
                            │        ▼              ▼             │       │
                            │  ┌─────────────────────────────┐   │       │
                            │  │        Service Layer        │   │       │
                            │  │                             │   │       │
                            │  │  UserService                │   │       │
                            │  │  InstanceService            │◄──┘       │
                            │  │  ChannelService             │           │
                            │  │  BillingService             │           │
                            │  │  NodeService (SSH control)  │           │
                            │  │  MonitoringService          │           │
                            │  └──────────┬──────────────────┘           │
                            │             │                              │
                            │    ┌────────┴────────┐                     │
                            │    ▼                  ▼                     │
                            │  ┌──────────┐  ┌───────────┐               │
                            │  │PostgreSQL│  │   Redis    │               │
                            │  │(Drizzle) │  │ (BullMQ)  │               │
                            │  └──────────┘  └───────────┘               │
                            └─────────────────────┬───────────────────────┘
                                                  │
                                          SSH / DigitalOcean API
                                                  │
                            ┌─────────────────────▼───────────────────────┐
                            │              VPS Nodes (Droplets)           │
                            │                                             │
                            │  ┌─────────────────────────────────────┐    │
                            │  │  openclaw gateway                   │    │
                            │  │  ├── WhatsApp channel               │    │
                            │  │  ├── Telegram channel               │    │
                            │  │  ├── Discord channel                │    │
                            │  │  └── ...                            │    │
                            │  └─────────────────────────────────────┘    │
                            └─────────────────────────────────────────────┘
```

---

## المكونات الرئيسية

### 1. API Router (Hono)

يستقبل كل طلبات HTTP من Layer 1:

```
/api/v1/auth/*          ← تسجيل + تسجيل دخول + JWT
/api/v1/users/*         ← إدارة الحساب
/api/v1/instances/*     ← إنشاء/حذف/تحكم بالـ instances
/api/v1/channels/*      ← ربط/فصل القنوات
/api/v1/billing/*       ← الاشتراكات والفواتير
/api/v1/usage/*         ← إحصائيات الاستخدام
/api/v1/admin/*         ← عمليات الإدارة
/api/v1/health          ← صحة النظام
```

### 2. Service Layer

كل خدمة مسؤولة عن مجال واحد:

| الخدمة | المسؤولية |
|--------|---------|
| **UserService** | التسجيل، الملف الشخصي، الأدوار، التعليق |
| **AuthService** | JWT، تجديد التوكن، التحقق من البريد |
| **InstanceService** | CRUD للـ instances، ربطها بالعقد |
| **NodeService** | SSH إلى السيرفرات، تنفيذ أوامر openclaw |
| **ChannelService** | ربط/فصل القنوات عبر openclaw CLI |
| **BillingService** | Stripe، الخطط، الفواتير |
| **UsageService** | تتبع الرسائل، حدود الاستخدام |
| **MonitoringService** | صحة العقد، uptime، تنبيهات |
| **AuditService** | سجل العمليات (من فعل ماذا ومتى) |

### 3. Node Controller (طبقتان: SSH + WebSocket RPC)

**اكتشاف مهم:** openclaw يملك بالفعل **WebSocket Gateway API مع 100+ RPC method**.
لذلك Layer 2 يستخدم **طريقتين** للتحكم:

#### الطريقة 1: SSH (للعمليات على مستوى النظام)
```
NodeService (SSH)
├── installOpenClaw(nodeId)       → npm i -g openclaw@latest
├── startGateway(nodeId, config)  → openclaw gateway run --bind loopback --port 18789
├── stopGateway(nodeId)           → pkill -f openclaw-gateway
├── updateOpenClaw(nodeId)        → npm i -g openclaw@latest && restart
├── getSystemResources(nodeId)    → CPU, RAM, Disk
└── setupNode(nodeId)             → first-time OS setup
```

#### الطريقة 2: WebSocket RPC (للعمليات على مستوى openclaw)
```
GatewayClient (WebSocket → ws://node-ip:18789/ws?token=...)
├── health                        → فحص الصحة
├── status                        → حالة الـ gateway
├── channels.status               → حالة كل القنوات
├── channels.logout               → تسجيل خروج قناة
├── config.get / config.set       → قراءة/تعديل الإعدادات
├── config.apply                  → تطبيق config كامل
├── sessions.list                 → قائمة الجلسات
├── sessions.reset                → إعادة تعيين جلسة
├── agents.*                      → إدارة الـ agents
├── send                          → إرسال رسالة
├── cron.*                        → إدارة المهام المجدولة
└── ... (100+ method إضافي)
```

**القاعدة:** SSH للتشغيل/الإيقاف/التثبيت. WebSocket لكل شيء آخر (أسرع وأكثر أماناً).

### 4. Job Queue (BullMQ + Redis)

المهام التي لا يجب أن تُنفّذ مباشرة (طويلة أو قابلة للفشل):

| Queue | المهام |
|-------|--------|
| `instance:provision` | إنشاء Droplet جديد + تثبيت openclaw |
| `instance:destroy` | إيقاف + حذف Droplet |
| `channel:link` | ربط قناة (قد يحتاج QR scan) |
| `billing:sync` | مزامنة مع Stripe |
| `monitoring:health` | فحص دوري لصحة كل العقد |
| `usage:aggregate` | تجميع إحصائيات الاستخدام يومياً |

### 5. WebSocket (Socket.io)

أحداث real-time للداشبورد:

```typescript
// الأحداث المُرسلة للـ Client
instance:status_changed    → { instanceId, status: 'running' | 'stopped' | ... }
instance:provision_progress → { instanceId, step: 3, total: 5, message: '...' }
channel:linked             → { instanceId, channelType: 'whatsapp', status: 'active' }
channel:qr_code            → { instanceId, qrBase64: '...' }
usage:threshold_warning    → { instanceId, usagePercent: 80 }
system:alert               → { type: 'node_down', nodeId, message: '...' }
```

---

## مبادئ التصميم

| المبدأ | التطبيق |
|--------|---------|
| **Layer 1 لا يتكلم مع VPS مباشرة** | كل شيء يمر عبر Layer 2 |
| **أوامر محدودة على SSH** | whitelist للأوامر المسموحة فقط |
| **Idempotent operations** | تكرار نفس الطلب لا يسبب مشاكل |
| **Queue للعمليات الطويلة** | provisioning/destroy عبر BullMQ |
| **Audit everything** | كل عملية حساسة تُسجَّل |
| **Graceful degradation** | إذا سقط node، الـ API يظل يعمل |

---

## تدفقات رئيسية

### تدفق إنشاء Instance جديد

```
User clicks "Create Instance"
        │
        ▼
Layer 1 → POST /api/v1/instances
        │
        ▼
Layer 2:
  1. يتحقق من صلاحية المستخدم وحدود خطته
  2. يُنشئ سجل instance في DB (status: provisioning)
  3. يُضيف مهمة لـ Queue: instance:provision
  4. يردّ بـ { instanceId, status: 'provisioning' }
        │
        ▼
Worker (instance:provision):
  1. يُنشئ Droplet جديد عبر DigitalOcean API
     (أو يأخذ من pool جاهز)
  2. ينتظر الـ Droplet يصبح active
  3. SSH → تثبيت openclaw
  4. SSH → تهيئة openclaw config
  5. SSH → تشغيل openclaw gateway
  6. يُحدّث DB: status → running
  7. يرسل WebSocket: instance:status_changed
        │
        ▼
Layer 1 يستقبل الحدث ويُحدّث الواجهة
```

### تدفق ربط قناة WhatsApp

```
User clicks "Link WhatsApp"
        │
        ▼
Layer 1 → POST /api/v1/instances/:id/channels/whatsapp
        │
        ▼
Layer 2:
  1. SSH → openclaw config set whatsapp.enabled true
  2. SSH → restart gateway
  3. يبدأ يراقب QR code generation
  4. WebSocket → channel:qr_code { qrBase64 }
        │
        ▼
Layer 1 يعرض QR Code للمستخدم
        │
        ▼
المستخدم يمسح الـ QR من WhatsApp
        │
        ▼
Layer 2:
  1. يكتشف أن القناة اتصلت
  2. يُحدّث DB: channel status → linked
  3. WebSocket → channel:linked
```

### تدفق Auth

```
POST /api/v1/auth/register
  → يُنشئ حساب + يرسل بريد تأكيد
  → يردّ بـ { message: 'check email' }

POST /api/v1/auth/verify-email
  → يفعّل الحساب

POST /api/v1/auth/login
  → يتحقق من credentials
  → يردّ بـ { accessToken, refreshToken }

POST /api/v1/auth/refresh
  → يُجدّد الـ accessToken

POST /api/v1/auth/logout
  → يُبطل الـ refreshToken
```

---

## الأمان

| الطبقة | الحماية |
|--------|---------|
| **Auth** | JWT مع access + refresh tokens |
| **Rate limiting** | حسب IP + حسب مستخدم |
| **SSH** | مفاتيح خاصة فقط (لا passwords) |
| **Commands** | whitelist — لا يُنفَّذ أمر عشوائي |
| **Input** | Zod validation لكل طلب |
| **CORS** | يسمح فقط لـ Layer 1 domains |
| **Audit** | كل عملية CRUD/admin مسجّلة |
| **Secrets** | env vars فقط — لا hardcoded |
