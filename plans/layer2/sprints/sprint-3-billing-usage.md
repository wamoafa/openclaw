# Sprint 3 — Billing + Usage Tracking

**المدة:** أسبوعان (10 أيام)
**الهدف:** Stripe متكامل — المستخدم يشترك ويدفع ويشوف استخدامه

---

## متى ينتهي Sprint 3 بنجاح؟

- [ ] المستخدم يختار خطة → يُحوَّل لـ Stripe Checkout → يدفع → اشتراكه يتفعّل
- [ ] تجاوز حد الخطة يمنع إنشاء instances/channels جديدة
- [ ] `GET /usage` → يعرض الاستخدام الشهري مع النسبة
- [ ] `GET /billing/invoices` → قائمة الفواتير مع PDF
- [ ] Webhook من Stripe يُحدّث الحالة تلقائياً (paid, failed, canceled)

---

## المهام

### S3-T-001 | Stripe Integration — Setup
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] `src/lib/stripe.ts` — Stripe client
- [ ] إنشاء Products + Prices في Stripe Dashboard:
  - Free plan (price = 0, no Stripe price)
  - Starter plan (monthly + yearly prices)
  - Pro plan (monthly + yearly prices)
- [ ] ربط `stripe_price_id_monthly` و `stripe_price_id_yearly` في جدول `plans`
- [ ] إنشاء Stripe Customer عند register أو أول اشتراك

**التبعيات:** S0-T-002
**الناتج:** Stripe جاهز مع Products/Prices

---

### S3-T-002 | Subscription Flow
**المسؤول:** Backend Dev
**الوقت:** 6 ساعات | **النقاط:** 5

- [ ] `src/services/billing.service.ts`:
  - `createCheckoutSession(userId, planId, billingCycle)` → Stripe Checkout Session
  - `getSubscription(userId)` → الاشتراك الحالي
  - `changePlan(userId, newPlanId)` → Stripe Subscription update
  - `cancelSubscription(userId)` → cancel at period end
  - `reactivateSubscription(userId)` → undo cancel
- [ ] Routes:
  - `POST /billing/subscribe` → Checkout URL
  - `GET /billing/subscription` → الاشتراك الحالي
  - `POST /billing/change-plan` → تغيير خطة
  - `POST /billing/cancel` → إلغاء
- [ ] Checkout success callback → update DB

**التبعيات:** S3-T-001
**الناتج:** تدفق الاشتراك كامل

---

### S3-T-003 | Stripe Webhooks
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] `POST /billing/webhook` — Stripe signature verification
- [ ] أحداث مدعومة:
  - `checkout.session.completed` → تفعيل الاشتراك
  - `invoice.paid` → إنشاء فاتورة في DB
  - `invoice.payment_failed` → تعليق الاشتراك
  - `customer.subscription.updated` → تحديث الخطة
  - `customer.subscription.deleted` → إلغاء الاشتراك
- [ ] Idempotent: نفس الحدث لا يُعالَج مرتين

**التبعيات:** S3-T-001
**الناتج:** DB متزامن مع Stripe دائماً

---

### S3-T-004 | Invoice Management
**المسؤول:** Backend Dev
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] `src/services/invoice.service.ts`:
  - `listInvoices(userId)` → الفواتير مع pagination
  - `getInvoicePdf(invoiceId)` → PDF URL من Stripe
- [ ] Routes:
  - `GET /billing/invoices`
  - `GET /billing/invoices/:id/pdf` → redirect to Stripe PDF

**التبعيات:** S3-T-003
**الناتج:** المستخدم يشوف فواتيره

---

### S3-T-005 | Plan Limits Enforcement
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] `src/services/plan.service.ts`:
  - `checkInstanceLimit(userId)` → هل يقدر ينشئ instance جديد؟
  - `checkChannelLimit(instanceId)` → هل يقدر يربط قناة جديدة؟
  - `checkMessageLimit(userId)` → هل تجاوز حد الرسائل؟
- [ ] Middleware: `/instances` و `/channels` يتحققون من الحدود قبل الإنشاء
- [ ] Error response واضح: `PLAN_LIMIT_REACHED` مع تفاصيل الحد

**التبعيات:** S1-T-008 (Plans), S3-T-002
**الناتج:** المستخدم لا يتجاوز حدود خطته

---

### S3-T-006 | Usage Tracking
**المسؤول:** Backend Dev
**الوقت:** 6 ساعات | **النقاط:** 5

- [ ] `src/services/usage.service.ts`:
  - `recordMessage(instanceId, direction)` → increment counter
  - `getUsageSummary(userId)` → ملخص شهري
  - `getUsageHistory(userId, months)` → آخر N أشهر
  - `getInstanceUsage(instanceId)` → استخدام instance محدد
- [ ] BullMQ recurring job: كل ساعة
  - لكل instance نشط: Gateway RPC → عدد الرسائل
  - تحديث `usage_records` في DB
- [ ] Daily aggregation job: تجميع يومي
- [ ] Routes:
  - `GET /usage`
  - `GET /usage/history`
  - `GET /usage/instances/:id`
- [ ] WebSocket: `usage:warning` عند 80% و 90% و 100%

**التبعيات:** S1-T-002 (Gateway Client)
**الناتج:** إحصائيات استخدام دقيقة

---

### S3-T-007 | Audit Logging
**المسؤول:** Backend Dev
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] `src/services/audit.service.ts`:
  - `log(actorId, action, resourceType, resourceId, details)`
- [ ] Middleware: يسجّل تلقائياً كل عمليات CRUD
- [ ] العمليات المسجّلة:
  - `user.register`, `user.login`, `user.suspend`
  - `instance.create`, `instance.delete`, `instance.start`, `instance.stop`
  - `channel.link`, `channel.unlink`
  - `subscription.create`, `subscription.cancel`, `subscription.change`
  - `admin.*` — كل عمليات الإدارة

**التبعيات:** S0
**الناتج:** كل عملية مسجّلة للمراجعة

---

### S3-T-008 | Tests — Billing + Usage
**المسؤول:** أي مطور
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] اختبار billing service (مع Stripe mock)
- [ ] اختبار webhook handling
- [ ] اختبار plan limits enforcement
- [ ] اختبار usage tracking

**التبعيات:** كل المهام أعلاه
**الناتج:** تغطية اختبارية للفوترة والاستخدام

---

## نقاط Sprint 3

| المهمة | النقاط |
|--------|--------|
| S3-T-001 (Stripe Setup) | 3 |
| S3-T-002 (Subscription Flow) | 5 |
| S3-T-003 (Webhooks) | 3 |
| S3-T-004 (Invoices) | 2 |
| S3-T-005 (Plan Limits) | 3 |
| S3-T-006 (Usage Tracking) | 5 |
| S3-T-007 (Audit Logging) | 2 |
| S3-T-008 (Tests) | 2 |
| **المجموع** | **25** |
