# Layer 2 — Tech Stack

## القرارات التقنية

### API Framework
**Hono**

| السبب | التفاصيل |
|-------|---------|
| خفيف وسريع | أسرع من Express بـ 10x+ |
| TypeScript-first | type-safe routing + middleware |
| متوافق مع Node.js | يشتغل على Bun و Node.js |
| Zod integration | `@hono/zod-validator` مدمج |
| OpenAPI | يولّد docs تلقائياً |

### المكتبات الأساسية

```
Layer 2 Tech Stack
├── Runtime
│   └── Bun (dev) / Node.js 22 (prod)
│
├── API Framework
│   ├── Hono                  ← HTTP routing + middleware
│   ├── @hono/zod-validator   ← request validation
│   └── @hono/swagger-ui      ← API docs (dev)
│
├── Database
│   ├── PostgreSQL 16         ← قاعدة البيانات الرئيسية
│   ├── Drizzle ORM           ← type-safe queries
│   ├── drizzle-kit           ← migrations
│   └── postgres.js           ← PostgreSQL driver
│
├── Auth
│   ├── jose                  ← JWT signing/verification
│   ├── bcrypt                ← password hashing
│   └── nodemailer / Resend   ← email verification
│
├── Job Queue
│   ├── BullMQ                ← background jobs
│   └── Redis (ioredis)       ← BullMQ backend + caching
│
├── Real-time
│   └── Socket.io             ← WebSocket events
│
├── Server Control
│   ├── node-ssh              ← SSH connections
│   └── digitalocean.js       ← DigitalOcean API (provisioning)
│
├── Billing
│   └── stripe                ← payments + subscriptions
│
├── Validation
│   └── Zod                   ← schemas مشتركة مع Layer 1
│
├── Monitoring
│   ├── pino                  ← structured logging
│   └── prom-client           ← Prometheus metrics (مستقبل)
│
├── Testing
│   ├── Vitest                ← unit + integration tests
│   └── supertest             ← HTTP endpoint testing
│
└── Dev Tools
    ├── ESLint + Oxlint
    ├── Oxfmt
    └── tsx                   ← TypeScript execution
```

## قرارات مقصودة

### لماذا Hono وليس Express/Fastify؟
- Express قديم وبطيء. Fastify جيد لكن Hono أخف وأسرع.
- Hono يدعم Zod validation مباشرة — لا حاجة لـ middleware إضافي.
- TypeScript-first بالكامل — كل route type-safe.
- سهل الانتقال لـ Cloudflare Workers مستقبلاً لو احتجنا.

### لماذا BullMQ وليس Agenda/Bee-Queue؟
- BullMQ هو المعيار — أكثر استخداماً وأفضل توثيقاً.
- يدعم retries، delays، priorities، rate limiting.
- يحتاج Redis — وRedis مفيد أيضاً للـ caching وSocket.io adapter.

### لماذا node-ssh وليس SSH2 مباشرة؟
- node-ssh أبسط API — لا نحتاج تحكم منخفض المستوى.
- يدعم exec commands + file transfer.
- SSH2 تحته — نقدر ننزل له لو احتجنا.

### لماذا jose وليس jsonwebtoken؟
- jose أحدث، يدعم Web Crypto API.
- لا يعتمد على native modules (أسرع في CI).
- يدعم JWK/JWKS لو احتجنا مفاتيح خارجية مستقبلاً.

---

## بنية المجلدات المقترحة

```
apps/api/                          ← (داخل monorepo)
├── src/
│   ├── index.ts                   ← Entry point (Hono app + server)
│   ├── app.ts                     ← Hono app creation + global middleware
│   │
│   ├── routes/
│   │   ├── auth.ts                ← /api/v1/auth/*
│   │   ├── users.ts               ← /api/v1/users/*
│   │   ├── instances.ts           ← /api/v1/instances/*
│   │   ├── channels.ts            ← /api/v1/channels/*
│   │   ├── billing.ts             ← /api/v1/billing/*
│   │   ├── usage.ts               ← /api/v1/usage/*
│   │   ├── admin.ts               ← /api/v1/admin/*
│   │   └── health.ts              ← /api/v1/health
│   │
│   ├── services/
│   │   ├── user.service.ts
│   │   ├── auth.service.ts
│   │   ├── instance.service.ts
│   │   ├── node.service.ts        ← SSH control
│   │   ├── channel.service.ts
│   │   ├── billing.service.ts
│   │   ├── usage.service.ts
│   │   ├── monitoring.service.ts
│   │   └── audit.service.ts
│   │
│   ├── middleware/
│   │   ├── auth.ts                ← JWT verification
│   │   ├── admin.ts               ← Admin role check
│   │   ├── rate-limit.ts          ← Rate limiting
│   │   └── error-handler.ts       ← Global error handling
│   │
│   ├── db/
│   │   ├── index.ts               ← Drizzle client
│   │   ├── schema.ts              ← كل الجداول
│   │   ├── migrations/            ← Drizzle migrations
│   │   └── seed.ts                ← بيانات تجريبية
│   │
│   ├── queues/
│   │   ├── index.ts               ← Queue setup + Redis connection
│   │   ├── instance.queue.ts      ← Provisioning/destroy jobs
│   │   ├── channel.queue.ts       ← Channel linking jobs
│   │   ├── billing.queue.ts       ← Billing sync jobs
│   │   └── monitoring.queue.ts    ← Health check jobs
│   │
│   ├── workers/
│   │   ├── instance.worker.ts     ← Processes instance jobs
│   │   ├── channel.worker.ts      ← Processes channel jobs
│   │   ├── billing.worker.ts      ← Processes billing jobs
│   │   └── monitoring.worker.ts   ← Processes health checks
│   │
│   ├── ws/
│   │   └── index.ts               ← Socket.io setup + event emitters
│   │
│   ├── lib/
│   │   ├── ssh.ts                 ← SSH connection pool
│   │   ├── digitalocean.ts        ← DO API wrapper
│   │   ├── stripe.ts              ← Stripe client
│   │   ├── email.ts               ← Email sending
│   │   ├── jwt.ts                 ← Token create/verify
│   │   └── config.ts              ← Environment config (Zod validated)
│   │
│   └── types/
│       ├── api.ts                 ← Request/Response types
│       └── models.ts              ← Business domain types
│
├── drizzle.config.ts
├── package.json
├── tsconfig.json
└── vitest.config.ts
```

---

## متطلبات الأداء

| المقياس | الهدف |
|---------|-------|
| Response time (CRUD) | < 100ms |
| Response time (SSH ops) | < 3s (async preferred) |
| Concurrent connections | 500+ |
| WebSocket connections | 200+ |
| Job processing (BullMQ) | < 60s for provisioning |
| Uptime | 99.5%+ |

---

## البيئات

```
local     → http://localhost:4000 (API) + :4001 (WebSocket)
staging   → http://45.55.253.17:4000
production → https://api.openclaw.ai
```

---

## متغيرات البيئة المطلوبة (.env)

```env
# Server
PORT=4000
WS_PORT=4001
NODE_ENV=development

# Database
DATABASE_URL=postgresql://openclaw:password@localhost:5432/openclaw_db

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_ACCESS_SECRET=<secret>
JWT_REFRESH_SECRET=<secret>
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# SSH (for node control)
SSH_PRIVATE_KEY_PATH=/etc/openclaw/ssh/id_ed25519

# DigitalOcean
DO_API_TOKEN=<token>
DO_SSH_KEY_ID=<key_id>
DO_DEFAULT_REGION=nyc1
DO_DEFAULT_SIZE=s-1vcpu-1gb

# Stripe
STRIPE_SECRET_KEY=<key>
STRIPE_WEBHOOK_SECRET=<key>

# Email
RESEND_API_KEY=<key>
EMAIL_FROM=noreply@openclaw.ai

# CORS
CORS_ORIGINS=http://localhost:3000,https://openclaw.ai

# Monitoring
LOG_LEVEL=info
```
