# Sprint 3 — Channels + Billing + Settings

**المدة:** أسبوعان (10 أيام)
**الهدف:** ربط القنوات، إدارة الاشتراك، إعدادات الحساب
**User Stories المغطاة:** US-D-007 → US-D-013

---

## متى ينتهي Sprint 3 بنجاح؟

- [ ] مستخدم يربط WhatsApp ويستقبل رسالة تأكيد
- [ ] مستخدم يربط Telegram bot ويختبره
- [ ] صفحة Billing تعرض الاشتراك الحالي مع Stripe
- [ ] مستخدم يغير اسمه وبريده من إعدادات الحساب

---

## المهام

### S3-T-001 | WhatsApp QR Connection
**المسؤول:** Frontend Dev + Backend
**الوقت التقديري:** 8 ساعات
**User Story:** US-D-007

- [ ] Component `WhatsAppConnector`:
  - جلب QR code من Core API (base64 image)
  - عرض QR مع تعليمات مصوّرة (3 خطوات)
  - Polling كل 5 ثوانٍ لحالة الاتصال
  - عداد تنازلي لانتهاء صلاحية QR (60 ثانية)
  - زر "تجديد QR" عند الانتهاء
- [ ] عند نجاح الاتصال: animation + رسالة نجاح + تحديث القناة

**الناتج:** WhatsApp connection flow

---

### S3-T-002 | Telegram Bot Connection
**المسؤول:** Frontend Dev
**الوقت التقديري:** 5 ساعات
**User Story:** US-D-008

- [ ] Component `TelegramConnector`:
  - حقل Bot Token مع placeholder مثل `123456:ABC-DEF1234...`
  - رابط "كيف أحصل على Token؟" → وثائق @BotFather
  - تحقق فوري من صحة الـ token (استدعاء API)
  - عرض اسم الـ bot بعد التحقق الناجح
  - تعليمات اختبار: "أرسل /start لـ @YourBotName"

**الناتج:** Telegram connection flow

---

### S3-T-003 | Channels List (في Instance)
**المسؤول:** Frontend Dev
**الوقت التقديري:** 4 ساعات
**User Story:** US-D-009

- [ ] قائمة القنوات المرتبطة بالـ Instance
- [ ] كل قناة: أيقونة، الاسم/الرقم، الحالة، تاريخ الربط
- [ ] badge خاص للقنوات ذات مشاكل
- [ ] زر "فصل" مع Dialog تأكيد
- [ ] زر "إضافة قناة" يفتح Modal لاختيار نوع القناة

**الناتج:** Channels management في Instance

---

### S3-T-004 | General Channels Page
**المسؤول:** Frontend Dev
**الوقت التقديري:** 3 ساعات

- [ ] `/dashboard/channels`: عرض كل القنوات عبر كل الـ instances
- [ ] فلتر حسب النوع والحالة
- [ ] نقر على قناة ينقل لـ Instance المرتبط

**الناتج:** Channels overview page

---

### S3-T-005 | Usage Page
**المسؤول:** Frontend Dev
**الوقت التقديري:** 5 ساعات
**User Story:** US-D-010

- [ ] ProgressBar: رسائل مستخدمة / الحد الكلي
- [ ] رسم بياني بسيط (Recharts LineChart): آخر 30 يوم
- [ ] تاريخ تجديد الحصة
- [ ] تنبيه عند 80% استخدام (banner أحمر فاتح)
- [ ] زر "ترقية الخطة" إذا كان على Free

**الناتج:** Usage page

---

### S3-T-006 | Billing Page + Stripe Integration
**المسؤول:** Frontend Dev + Backend
**الوقت التقديري:** 10 ساعات
**User Story:** US-D-011

- [ ] عرض الخطة الحالية + سعرها + تاريخ التجديد
- [ ] تاريخ الفواتير السابقة مع رابط PDF (من Stripe)
- [ ] زر "ترقية الخطة" → Pricing comparison popup
- [ ] زر "إدارة طريقة الدفع" → Stripe Customer Portal redirect
- [ ] زر "إلغاء الاشتراك" مع Dialog يشرح ما سيحدث
- [ ] Stripe webhook handler (في Next.js API route):
  - `invoice.paid` → تحديث حالة الاشتراك
  - `customer.subscription.deleted` → تخفيض للـ Free

**الناتج:** Billing كامل مع Stripe

---

### S3-T-007 | Account Settings
**المسؤول:** Frontend Dev
**الوقت التقديري:** 6 ساعات
**User Story:** US-D-012

- [ ] Tab "المعلومات الشخصية": اسم + بريد + avatar
- [ ] Tab "الأمان": تغيير كلمة المرور
- [ ] Tab "الإشعارات": تفضيلات البريد
- [ ] Tab "اللغة": EN / AR toggle
- [ ] كل تغيير يحفظ بزر "حفظ" منفصل (لا auto-save)
- [ ] رسالة تأكيد بريد عند تغيير البريد

**الناتج:** Settings page

---

### S3-T-008 | Delete Account
**المسؤول:** Frontend Dev
**الوقت التقديري:** 3 ساعات
**User Story:** US-D-013

- [ ] زر "حذف الحساب" في أسفل Settings (باللون الأحمر)
- [ ] Dialog يظهر قائمة ما سيُحذف
- [ ] يطلب كلمة المرور للتأكيد
- [ ] بعد الحذف: logout + redirect لـ Landing

**الناتج:** Delete Account flow

---

### S3-T-009 | Notifications System
**المسؤول:** Frontend Dev
**الوقت التقديري:** 5 ساعات

- [ ] Notification bell في Top Bar مع badge عدد غير المقروء
- [ ] Dropdown يعرض آخر 10 إشعارات
- [ ] أنواع الإشعارات: Instance status، channel status، billing alerts
- [ ] تحديد مقروء عند النقر
- [ ] WebSocket للإشعارات الفورية

**الناتج:** Notification system

---

### S3-T-010 | اختبارات Sprint 3
**المسؤول:** كل المطورين
**الوقت التقديري:** 4 ساعات

- [ ] unit: Stripe webhook handler
- [ ] unit: Channel connection validators
- [ ] integration: WhatsApp QR polling
- [ ] e2e (Playwright): Billing upgrade flow

**الناتج:** test coverage للكود الجديد

---

## نقاط Sprint 3

| المهمة | النقاط |
|--------|--------|
| S3-T-001 WhatsApp | 5 |
| S3-T-002 Telegram | 3 |
| S3-T-003 Channels List | 3 |
| S3-T-004 Channels Page | 2 |
| S3-T-005 Usage | 3 |
| S3-T-006 Billing + Stripe | 7 |
| S3-T-007 Settings | 4 |
| S3-T-008 Delete Account | 2 |
| S3-T-009 Notifications | 3 |
| S3-T-010 Tests | 3 |
| **المجموع** | **35** |
