# Sprint 4 — Admin Panel

**المدة:** أسبوعان (10 أيام)
**الهدف:** Admin Panel كامل: Users, Instances, Billing, System Health
**User Stories المغطاة:** US-A-001 → US-A-012

---

## متى ينتهي Sprint 4 بنجاح؟

- [ ] مسؤول يرى KPIs النظام فور الدخول
- [ ] يستطيع البحث عن مستخدم وتعليق حسابه
- [ ] لوحة الإيرادات تعرض MRR الصحيح
- [ ] كل الإجراءات الإدارية مسجّلة في Audit Log

---

## المهام

### S4-T-001 | Admin Route Protection
**المسؤول:** Backend / Auth Dev
**الوقت التقديري:** 3 ساعات

- [ ] middleware يتحقق من `role === 'admin'` في JWT
- [ ] لو غير مسؤول: redirect لـ `/dashboard` مع رسالة "لا صلاحية"
- [ ] API routes الـ admin تتحقق من الـ role أيضاً (server-side)
- [ ] اختبار: محاولة الوصول بـ user عادي تُرفض

**الناتج:** Admin routes محمية

---

### S4-T-002 | Admin Overview Page
**المسؤول:** Frontend Dev
**الوقت التقديري:** 7 ساعات
**User Story:** US-A-001

- [ ] KPI Cards:
  - مستخدمون نشطون (اليوم / الشهر)
  - Instances تشتغل الآن
  - رسائل اليوم
  - MRR الشهر الحالي
- [ ] LineChart: نمو المستخدمين آخر 30 يوم (Recharts)
- [ ] قائمة آخر 10 مستخدمين سجّلوا
- [ ] تنبيهات نظام فعّالة

**الناتج:** Admin Dashboard Overview

---

### S4-T-003 | Users List
**المسؤول:** Frontend Dev
**الوقت التقديري:** 6 ساعات
**User Story:** US-A-002

- [ ] جدول بـ DataTable (shadcn + TanStack Table):
  - columns: الاسم، البريد، الخطة، تاريخ التسجيل، الحالة
  - sorting لكل column
  - pagination server-side
- [ ] فلتر: الخطة + الحالة + تاريخ التسجيل (range)
- [ ] بحث real-time بالبريد أو الاسم (debounced 300ms)
- [ ] زر "تصدير CSV"

**الناتج:** Users List مع كل أدوات الإدارة

---

### S4-T-004 | User Detail Page
**المسؤول:** Frontend Dev
**الوقت التقديري:** 7 ساعات
**User Story:** US-A-003, US-A-004

- [ ] Header: معلومات المستخدم + حالته الحالية
- [ ] Tab "الـ Instances": قائمة instances المستخدم
- [ ] Tab "الاستخدام": رسم بياني + إحصائيات
- [ ] Tab "الفواتير": تاريخ المدفوعات
- [ ] Tab "النشاط": login history
- [ ] Actions Panel (يمين الصفحة):
  - تعليق / إلغاء تعليق
  - تغيير الخطة يدوياً
  - منح تجربة ممتدة
  - إرسال بريد مباشر

**الناتج:** User Detail page

---

### S4-T-005 | Suspend/Unsuspend User
**المسؤول:** Frontend Dev
**الوقت التقديري:** 3 ساعات
**User Story:** US-A-004

- [ ] Dialog تعليق: حقل "سبب التعليق" (مطلوب)
- [ ] تأكيد التعليق → API call → تحديث حالة المستخدم
- [ ] إيقاف كل instances المستخدم تلقائياً
- [ ] إشعار بريد للمستخدم عن التعليق
- [ ] Dialog "إلغاء تعليق" مع نفس التدفق
- [ ] كل إجراء يُسجّل في Audit Log (S4-T-011)

**الناتج:** Suspend/Unsuspend flow

---

### S4-T-006 | Admin Instances View
**المسؤول:** Frontend Dev
**الوقت التقديري:** 5 ساعات
**User Story:** US-A-006, US-A-007

- [ ] جدول: المعرّف، المالك، المنطقة، الحالة، CPU/RAM
- [ ] فلتر الحالة والمنطقة والمالك
- [ ] إجمالي instances نشطة في header
- [ ] زر "إيقاف" مع Dialog سبب + إشعار للمالك
- [ ] زر "إعادة تشغيل"

**الناتج:** Admin Instances management

---

### S4-T-007 | Revenue Dashboard
**المسؤول:** Frontend Dev + Backend
**الوقت التقديري:** 8 ساعات
**User Story:** US-A-008

- [ ] KPIs: MRR، نمو MRR%، Churn Rate، متوسط Revenue/User
- [ ] AreaChart: MRR آخر 12 شهر
- [ ] PieChart: توزيع حسب الخطة
- [ ] جدول المشتركين الجدد هذا الشهر
- [ ] جدول الملغين هذا الشهر
- [ ] بيانات حقيقية من Stripe API (backend يجلبها)

**الناتج:** Revenue Dashboard

---

### S4-T-008 | Invoices Admin View
**المسؤول:** Frontend Dev
**الوقت التقديري:** 4 ساعات
**User Story:** US-A-009

- [ ] قائمة الفواتير: المستخدم، المبلغ، التاريخ، الحالة
- [ ] فلتر: الحالة + الفترة الزمنية
- [ ] رابط لـ Stripe dashboard لكل فاتورة
- [ ] زر "استرداد" مع Dialog تأكيد + قيمة الاسترداد

**الناتج:** Invoices admin view

---

### S4-T-009 | System Health Page
**المسؤول:** Frontend Dev + Backend
**الوقت التقديري:** 6 ساعات
**User Story:** US-A-010

- [ ] بطاقات الحالة: Core API، Broker، Database، Queue
- [ ] Uptime percentage (آخر 30 يوم لكل service)
- [ ] متوسط API latency (آخر 24 ساعة)
- [ ] جدول آخر 20 خطأ مع: الوقت، النوع، التفاصيل
- [ ] زر "تحديث" يعيد جلب البيانات
- [ ] تلوين حسب الحالة: 🟢 سليم / 🟡 تحذير / 🔴 خطأ

**الناتج:** System Health monitoring

---

### S4-T-010 | Announcements
**المسؤول:** Frontend Dev
**الوقت التقديري:** 5 ساعات
**User Story:** US-A-011

- [ ] فورم إنشاء إعلان: العنوان + المحتوى (Markdown editor بسيط) + الاستهداف
- [ ] معاينة الإعلان قبل الإرسال
- [ ] قائمة الإعلانات السابقة مع: عدد المستلمين، عدد القراءة
- [ ] في Dashboard المستخدم: banner لعرض الإعلانات النشطة

**الناتج:** Announcements system

---

### S4-T-011 | Audit Log
**المسؤول:** Frontend Dev + Backend
**الوقت التقديري:** 5 ساعات
**User Story:** US-A-012

- [ ] Backend: تسجيل كل admin action في `audit_log` table
- [ ] صفحة Audit Log:
  - جدول: المسؤول، الإجراء، الهدف، الوقت
  - بحث وفلترة
  - تصدير CSV
- [ ] لا يمكن حذف أو تعديل السجلات (API ترفض DELETE/PUT)

**الناتج:** Audit Log كامل

---

### S4-T-012 | اختبارات Sprint 4
**المسؤول:** كل المطورين
**الوقت التقديري:** 4 ساعات

- [ ] unit: Admin middleware (role check)
- [ ] unit: Revenue calculations
- [ ] integration: Suspend user → instances stop
- [ ] e2e: Admin searches user → suspends → verifies

**الناتج:** tests للـ Admin Panel

---

## نقاط Sprint 4

| المهمة | النقاط |
|--------|--------|
| S4-T-001 Route Protection | 2 |
| S4-T-002 Overview | 5 |
| S4-T-003 Users List | 4 |
| S4-T-004 User Detail | 5 |
| S4-T-005 Suspend | 2 |
| S4-T-006 Admin Instances | 3 |
| S4-T-007 Revenue Dashboard | 5 |
| S4-T-008 Invoices | 3 |
| S4-T-009 System Health | 4 |
| S4-T-010 Announcements | 3 |
| S4-T-011 Audit Log | 4 |
| S4-T-012 Tests | 3 |
| **المجموع** | **43** |
