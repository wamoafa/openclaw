# Sprint 0 — Foundation Setup

**المدة:** أسبوع واحد (5 أيام)
**الهدف:** مشروع Hono API جاهز مع PostgreSQL + Auth + Health endpoint يعمل على `45.55.253.17`

---

## متى ينتهي Sprint 0 بنجاح؟

- [ ] API يشتغل على `http://45.55.253.17:4000/api/v1/health`
- [ ] PostgreSQL يعمل + كل الجداول موجودة (migration)
- [ ] `POST /auth/register` → ينشئ مستخدم في DB
- [ ] `POST /auth/login` → يرجع JWT
- [ ] Middleware يحمي `/api/v1/users/me`
- [ ] Redis يعمل (للـ queue لاحقاً)
- [ ] CI يمرّ: build + lint + typecheck + tests

---

## المهام

### S0-T-001 | إعداد مشروع Hono + TypeScript
**المسؤول:** Backend Lead
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] إنشاء `apps/api/` في الـ monorepo
- [ ] `package.json` مع dependencies: `hono`, `drizzle-orm`, `postgres`, `zod`, `jose`
- [ ] `tsconfig.json` مع strict mode
- [ ] `src/index.ts` — Hono app يسمع على `PORT`
- [ ] `src/app.ts` — إنشاء الـ app مع global middleware (cors, error handler, logger)
- [ ] `GET /api/v1/health` يردّ `{ status: "ok" }`

**التبعيات:** لا شيء
**الناتج:** `bun run src/index.ts` → server يعمل

---

### S0-T-002 | إعداد PostgreSQL + Drizzle
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] تثبيت PostgreSQL 16 على `45.55.253.17` (أو التأكد من وجوده)
- [ ] إنشاء database: `openclaw_db`
- [ ] إنشاء user: `openclaw` مع صلاحيات
- [ ] `src/db/index.ts` — Drizzle client مع connection pool
- [ ] `src/db/schema.ts` — كل الجداول (users, refresh_tokens, email_verifications, nodes, instances, channels, plans, subscriptions, invoices, usage_records, audit_logs, announcements, system_settings)
- [ ] `drizzle.config.ts` + أول migration
- [ ] `src/db/seed.ts` — بيانات أولية (admin user, default plans)
- [ ] اختبار الاتصال من local → server

**التبعيات:** S0-T-001
**الناتج:** `bunx drizzle-kit migrate` → جداول جاهزة

---

### S0-T-003 | Auth — Registration + Login + JWT
**المسؤول:** Backend Dev
**الوقت:** 6 ساعات | **النقاط:** 5

- [ ] `src/lib/jwt.ts` — sign/verify access + refresh tokens (jose)
- [ ] `src/lib/password.ts` — hash/verify (bcrypt)
- [ ] `src/services/auth.service.ts`:
  - `register(email, password, name)` → hash + insert + send verification email
  - `login(email, password)` → verify + generate tokens
  - `refreshToken(token)` → verify + rotate
  - `logout(refreshToken)` → revoke
  - `verifyEmail(token)` → mark verified
- [ ] `src/routes/auth.ts`:
  - `POST /auth/register`
  - `POST /auth/login`
  - `POST /auth/refresh`
  - `POST /auth/logout`
  - `POST /auth/verify-email`
- [ ] `src/middleware/auth.ts` — JWT verification middleware
- [ ] `src/middleware/admin.ts` — admin role check
- [ ] Zod schemas لكل request

**التبعيات:** S0-T-002
**الناتج:** auth كامل يعمل مع JWT

---

### S0-T-004 | User Profile Endpoint
**المسؤول:** Backend Dev
**الوقت:** 2 ساعة | **النقاط:** 2

- [ ] `src/services/user.service.ts`:
  - `getProfile(userId)`
  - `updateProfile(userId, data)`
  - `changePassword(userId, oldPass, newPass)`
- [ ] `src/routes/users.ts`:
  - `GET /users/me`
  - `PATCH /users/me`
  - `PATCH /users/me/password`

**التبعيات:** S0-T-003
**الناتج:** مستخدم مسجّل يقدر يشوف ويعدّل ملفه

---

### S0-T-005 | Redis Setup
**المسؤول:** DevOps / Backend Dev
**الوقت:** 1 ساعة | **النقاط:** 1

- [ ] تثبيت Redis على `45.55.253.17`
- [ ] `src/lib/redis.ts` — ioredis client
- [ ] اختبار الاتصال
- [ ] إضافة `REDIS_URL` لـ `.env.example`

**التبعيات:** لا شيء
**الناتج:** Redis جاهز

---

### S0-T-006 | Error Handling + Logging
**المسؤول:** Backend Dev
**الوقت:** 2 ساعة | **النقاط:** 2

- [ ] `src/middleware/error-handler.ts` — global error handler بصيغة موحّدة
- [ ] `src/lib/errors.ts` — custom error classes (AppError, NotFoundError, ValidationError, etc.)
- [ ] `src/lib/logger.ts` — pino logger مع structured logging
- [ ] كل الـ routes تستخدم الأخطاء المخصصة

**التبعيات:** S0-T-001
**الناتج:** أخطاء واضحة وسجل منظّم

---

### S0-T-007 | Config + Environment Validation
**المسؤول:** Backend Dev
**الوقت:** 1 ساعة | **النقاط:** 1

- [ ] `src/lib/config.ts` — Zod schema لكل env vars
- [ ] `.env.example` مع كل المتغيرات
- [ ] التطبيق يرفض التشغيل إذا متغير مطلوب ناقص

**التبعيات:** S0-T-001
**الناتج:** لا أخطاء config وقت التشغيل

---

### S0-T-008 | Vitest + أول اختبارات
**المسؤول:** أي مطور
**الوقت:** 2 ساعة | **النقاط:** 2

- [ ] `vitest.config.ts` لـ `apps/api`
- [ ] اختبار auth service (register, login, refresh)
- [ ] اختبار JWT middleware
- [ ] اختبار health endpoint

**التبعيات:** S0-T-003
**الناتج:** test suite تشتغل مع CI

---

## نقاط Sprint 0

| المهمة | النقاط |
|--------|--------|
| S0-T-001 (Hono setup) | 2 |
| S0-T-002 (PostgreSQL + Drizzle) | 3 |
| S0-T-003 (Auth) | 5 |
| S0-T-004 (User profile) | 2 |
| S0-T-005 (Redis) | 1 |
| S0-T-006 (Error handling) | 2 |
| S0-T-007 (Config) | 1 |
| S0-T-008 (Tests) | 2 |
| **المجموع** | **18** |
