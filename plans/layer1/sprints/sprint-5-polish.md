# Sprint 5 — Polish, QA, Performance, i18n

**المدة:** أسبوع واحد (5 أيام)
**الهدف:** تلميع، اختبار نهائي، Arabic support، جهوزية للإطلاق
**متطلب:** Sprint 0→4 مكتملة بنجاح

---

## متى ينتهي Sprint 5 بنجاح؟

- [ ] التطبيق بالكامل يعمل بدون console errors
- [ ] Arabic UI محترم (RTL، خط مناسب)
- [ ] Lighthouse > 90 على كل الصفحات الرئيسية
- [ ] E2E tests تمرر على CI
- [ ] Accessibility score > 80 (axe DevTools)
- [ ] تقرير QA يؤكد: لا P0/P1 bugs مفتوحة

---

## المهام

### S5-T-001 | RTL + Arabic Language Support
**المسؤول:** Frontend Lead
**الوقت التقديري:** 8 ساعات

- [ ] إعداد `next-intl` أو `react-i18next`
- [ ] ملفات الترجمة: `messages/en.json` و `messages/ar.json`
- [ ] نقل كل النصوص الـ hardcoded لملفات الترجمة
- [ ] `dir="rtl"` على `<html>` عند العربية
- [ ] Tailwind RTL support (`start-*`/`end-*` بدل `left-*`/`right-*`)
- [ ] خط عربي مناسب (Noto Sans Arabic أو Cairo)
- [ ] اختبار: السايدبار، الفورمز، الجداول في RTL

**الناتج:** دعم عربي كامل

---

### S5-T-002 | Accessibility (a11y)
**المسؤول:** Frontend Dev
**الوقت التقديري:** 6 ساعات

- [ ] فحص كامل بـ axe DevTools على صفحات رئيسية
- [ ] إصلاح: labels ناقصة، contrast، focus management
- [ ] keyboard navigation يعمل في الداشبورد
- [ ] `aria-live` regions للإشعارات الديناميكية
- [ ] Skip link "انتقل للمحتوى الرئيسي"

**الناتج:** a11y score > 80

---

### S5-T-003 | Performance Audit + Fix
**المسؤول:** Frontend Lead
**الوقت التقديري:** 6 ساعات

- [ ] قياس Lighthouse على: Landing, Dashboard Home, Instance Detail
- [ ] إصلاح: Large Images، Unused JS، CLS
- [ ] Code splitting للـ Admin Panel (dynamic import)
- [ ] React.memo لـ components غير ضرورية في Re-render
- [ ] تأكيد: LCP < 2.5s على Landing، TTI < 3.5s على Dashboard

**الناتج:** Performance targets محقّقة

---

### S5-T-004 | QA Bug Bash
**المسؤول:** كل الفريق (يوم واحد)
**الوقت التقديري:** 8 ساعات

- [ ] جلسة QA منظّمة: كل مطور يختبر قسماً مختلفاً
- [ ] تسجيل كل bug في GitHub Issues مع:
  - Steps to reproduce
  - Expected vs Actual
  - Screenshot/Video
  - Priority (P0/P1/P2)
- [ ] إصلاح كل P0 و P1 في نفس Sprint

**الناتج:** لا P0/P1 bugs مفتوحة قبل الإطلاق

---

### S5-T-005 | E2E Tests (Playwright) الشاملة
**المسؤول:** QA / Frontend Dev
**الوقت التقديري:** 6 ساعات

- [ ] Scenario 1: Visitor → Signup → Email Verify → Onboarding → Instance Created
- [ ] Scenario 2: User → Connect WhatsApp → Test Message
- [ ] Scenario 3: User → Upgrade Plan → Billing Updated
- [ ] Scenario 4: Admin → Search User → Suspend → Verify Logout
- [ ] Scenario 5: Admin → View Revenue → Export CSV

**الناتج:** E2E test suite تمرر على CI

---

### S5-T-006 | Error Pages
**المسؤول:** Frontend Dev
**الوقت التقديري:** 3 ساعات

- [ ] صفحة 404 مع رابط للرئيسية
- [ ] صفحة 500 مع رابط "تواصل مع الدعم"
- [ ] صفحة "انتهت الجلسة" مع redirect لـ login
- [ ] صفحة Maintenance (يمكن تفعيلها عند الصيانة)

**الناتج:** Error pages احترافية

---

### S5-T-007 | Security Headers + Final Review
**المسؤول:** Frontend Lead
**الوقت التقديري:** 3 ساعات

- [ ] `next.config.ts` headers:
  - `Content-Security-Policy`
  - `X-Frame-Options: DENY`
  - `X-Content-Type-Options: nosniff`
  - `Referrer-Policy`
- [ ] مراجعة: لا secrets في client bundle
- [ ] مراجعة: لا console.log في production

**الناتج:** Security headers مُطبّقة

---

### S5-T-008 | Documentation الداخلية
**المسؤول:** Frontend Lead
**الوقت التقديري:** 3 ساعات

- [ ] `apps/web/README.md`: كيف تشغّل المشروع
- [ ] `apps/web/ARCHITECTURE.md`: هيكل المجلدات والقرارات
- [ ] Comment لكل hook غير واضح
- [ ] تحديث `plans/layer1/` بأي تغييرات حدثت أثناء التنفيذ

**الناتج:** توثيق داخلي محدّث

---

## نقاط Sprint 5

| المهمة | النقاط |
|--------|--------|
| S5-T-001 RTL + i18n | 5 |
| S5-T-002 a11y | 4 |
| S5-T-003 Performance | 4 |
| S5-T-004 QA Bug Bash | 5 |
| S5-T-005 E2E Tests | 4 |
| S5-T-006 Error Pages | 2 |
| S5-T-007 Security | 2 |
| S5-T-008 Docs | 2 |
| **المجموع** | **28** |
