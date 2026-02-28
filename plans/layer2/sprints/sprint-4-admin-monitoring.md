# Sprint 4 — Admin API + Monitoring

**المدة:** أسبوعان (10 أيام)
**الهدف:** لوحة إدارة كاملة — مراقبة النظام، إدارة المستخدمين، إحصائيات الإيرادات

---

## متى ينتهي Sprint 4 بنجاح؟

- [ ] Admin يشوف كل المستخدمين مع فلترة وبحث
- [ ] Admin يعلّق/يفعّل حساب مستخدم
- [ ] Admin يشوف صحة كل الـ nodes + instances
- [ ] Admin يشوف ملخص الإيرادات (MRR, الاشتراكات النشطة)
- [ ] Admin يشوف سجل المراجعة (audit logs)
- [ ] Admin ينشر إعلانات للمستخدمين
- [ ] Health monitoring تلقائي مع تنبيهات

---

## المهام

### S4-T-001 | Admin — User Management API
**المسؤول:** Backend Dev
**الوقت:** 6 ساعات | **النقاط:** 5

- [ ] `src/services/admin.service.ts`:
  - `listUsers(filters, pagination)` → كل المستخدمين مع بحث/فلترة
  - `getUserDetails(userId)` → تفاصيل كاملة (instances, channels, usage, plan)
  - `updateUser(userId, data)` → تعديل role/plan
  - `suspendUser(userId, reason)` → تعليق + إيقاف كل instances
  - `unsuspendUser(userId)` → إلغاء التعليق + إعادة تشغيل
- [ ] Routes في `src/routes/admin.ts`:
  - `GET /admin/users?page=1&limit=20&search=&status=&role=&plan=`
  - `GET /admin/users/:id`
  - `PATCH /admin/users/:id`
  - `POST /admin/users/:id/suspend`
  - `POST /admin/users/:id/unsuspend`
- [ ] Suspend cascade: يوقف كل instances + يفصل كل channels

**التبعيات:** S0-T-003 (Auth + Admin middleware)
**الناتج:** Admin يدير المستخدمين بالكامل

---

### S4-T-002 | Admin — Instance Overview
**المسؤول:** Backend Dev
**الوقت:** 3 ساعات | **النقاط:** 3

- [ ] Routes:
  - `GET /admin/instances?page=1&status=&userId=&nodeId=`
  - `GET /admin/instances/:id` → مع owner info + node info
  - `POST /admin/instances/:id/force-stop`
- [ ] Join مع users + nodes للعرض الكامل

**التبعيات:** S1-T-004
**الناتج:** Admin يشوف كل instances في النظام

---

### S4-T-003 | Admin — Node Health Monitoring
**المسؤول:** Backend Dev (Systems)
**الوقت:** 6 ساعات | **النقاط:** 5

- [ ] `src/services/monitoring.service.ts`:
  - `checkNodeHealth(nodeId)` → SSH ping + process check + resources
  - `checkAllNodes()` → فحص شامل
  - `getSystemHealth()` → API + DB + Redis + Nodes
- [ ] BullMQ recurring job: كل 60 ثانية
  - فحص كل node: SSH reachable? Gateway running? Memory ok?
  - تحديث `nodes.last_health_at` + `nodes.status`
  - إذا node سقط → WebSocket: `system:alert` لـ admin room
- [ ] Routes:
  - `GET /admin/system/health` → صحة كل الخدمات
  - `POST /admin/nodes/:id/health-check` → فحص فوري
- [ ] Alert thresholds:
  - Node unreachable → alert
  - CPU > 90% → warning
  - Memory > 85% → warning
  - Disk > 90% → critical

**التبعيات:** S1-T-001 (Node Service)
**الناتج:** مراقبة مستمرة مع تنبيهات

---

### S4-T-004 | Admin — Revenue Dashboard API
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] `src/services/revenue.service.ts`:
  - `getSummary()` → total users, active subscriptions, MRR, revenue this month
  - `getMRRHistory(months)` → MRR chart data (آخر 12 شهر)
  - `getSubscriptionsByPlan()` → breakdown per plan
  - `getRecentTransactions(limit)` → آخر المعاملات
- [ ] Routes:
  - `GET /admin/revenue/summary`
  - `GET /admin/revenue/mrr?months=12`
- [ ] SQL aggregations:
  - MRR = sum of active monthly subscriptions
  - Revenue = sum of paid invoices this month

**التبعيات:** S3 (Billing)
**الناتج:** Admin يشوف إحصائيات مالية

---

### S4-T-005 | Admin — Audit Logs API
**المسؤول:** Backend Dev
**الوقت:** 2 ساعة | **النقاط:** 2

- [ ] Routes:
  - `GET /admin/audit-logs?page=1&action=&actorId=&resourceType=&from=&to=`
- [ ] فلترة متقدمة: حسب الفاعل، النوع، الفترة الزمنية
- [ ] كل سجل يُظهر: من، ماذا فعل، على أي مورد، متى، من أي IP

**التبعيات:** S3-T-007 (Audit logging)
**الناتج:** Admin يراجع كل العمليات

---

### S4-T-006 | Admin — Announcements
**المسؤول:** Backend Dev
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] `src/services/announcement.service.ts`:
  - `create(data)`, `list()`, `update(id, data)`, `delete(id)`
  - `getActive()` → الإعلانات النشطة حالياً
- [ ] Routes:
  - `GET /admin/announcements`
  - `POST /admin/announcements`
  - `PATCH /admin/announcements/:id`
  - `DELETE /admin/announcements/:id`
- [ ] User-facing endpoint: `GET /announcements/active` (public for logged-in users)
- [ ] WebSocket: `announcement:new` عند إنشاء إعلان

**التبعيات:** S0
**الناتج:** Admin يُدير الإعلانات

---

### S4-T-007 | Email Notifications
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] `src/lib/email.ts` — Resend client
- [ ] Templates:
  - Welcome email (بعد التسجيل)
  - Email verification
  - Password reset
  - Subscription confirmed
  - Payment failed
  - Instance down alert
  - Usage warning (80%, 90%)
- [ ] BullMQ queue: `email` — لا ترسل مباشرة

**التبعيات:** S0
**الناتج:** إشعارات بريدية تلقائية

---

### S4-T-008 | Rate Limiting + Security Hardening
**المسؤول:** Backend Dev
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] `src/middleware/rate-limit.ts`:
  - Auth endpoints: 10 req/min per IP
  - General API: 100 req/min per user
  - Admin API: 200 req/min per user
- [ ] CORS: يسمح فقط لـ origins محددة
- [ ] Helmet-equivalent headers
- [ ] Input sanitization review

**التبعيات:** S0
**الناتج:** API محمي ضد الاستغلال

---

### S4-T-009 | API Documentation (OpenAPI)
**المسؤول:** أي مطور
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] Hono OpenAPI integration (`@hono/swagger-ui`)
- [ ] كل route موثّق مع request/response schemas
- [ ] Swagger UI على `/api/docs` (dev فقط)
- [ ] Export OpenAPI spec لـ Layer 1 team

**التبعيات:** كل الـ routes
**الناتج:** توثيق تفاعلي لكل الـ API

---

### S4-T-010 | Final Integration Tests + Load Test
**المسؤول:** QA / Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] E2E test: register → subscribe → create instance → link channel → usage → invoice
- [ ] Admin tests: list users, suspend, health check
- [ ] Load test: 100 concurrent API requests
- [ ] WebSocket test: 50 concurrent connections مع events

**التبعيات:** كل المهام
**الناتج:** API مستقر وجاهز للإنتاج

---

## نقاط Sprint 4

| المهمة | النقاط |
|--------|--------|
| S4-T-001 (User Management) | 5 |
| S4-T-002 (Instance Overview) | 3 |
| S4-T-003 (Health Monitoring) | 5 |
| S4-T-004 (Revenue Dashboard) | 3 |
| S4-T-005 (Audit Logs) | 2 |
| S4-T-006 (Announcements) | 2 |
| S4-T-007 (Email) | 3 |
| S4-T-008 (Security) | 2 |
| S4-T-009 (API Docs) | 2 |
| S4-T-010 (Tests) | 3 |
| **المجموع** | **30** |
