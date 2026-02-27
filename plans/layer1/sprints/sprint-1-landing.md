# Sprint 1 — Landing Page

**المدة:** أسبوعان (10 أيام)
**الهدف:** Landing Page كاملة جاهزة للإطلاق، تشمل Hero, Features, Pricing, Auth pages
**User Stories المغطاة:** US-L-001 إلى US-L-009

---

## متى ينتهي Sprint 1 بنجاح؟

- [ ] صفحة Landing كاملة ومتجاوبة (375px → 1440px)
- [ ] Lighthouse Score > 90 على Landing
- [ ] صفحات Login/Signup/Forgot Password تعمل
- [ ] كل الـ copy بدون placeholders تقنية مكشوفة للمستخدم
- [ ] تمرير CI بدون أخطاء

---

## المهام

### S1-T-001 | Hero Section
**المسؤول:** Frontend Dev + Copywriter
**الوقت التقديري:** 6 ساعات
**User Story:** US-L-001

- [ ] عنوان رئيسي يصف الفائدة (مش التقنية)
- [ ] جملة وصفية واحدة
- [ ] زرا CTA: "ابدأ مجاناً" + "شاهد كيف يعمل"
- [ ] صورة Hero (Device mockup أو screenshot)
- [ ] Animation بسيطة عند التحميل (fade-in)
- [ ] responsive: يتكيف من mobile إلى desktop

**الناتج:** Hero Section مكتملة

---

### S1-T-002 | Channels Section
**المسؤول:** Frontend Dev
**الوقت التقديري:** 4 ساعات
**User Story:** US-L-002

- [ ] شبكة أيقونات القنوات (WhatsApp, Telegram, Discord, iMessage, Signal, Slack)
- [ ] badge "قريباً" للقنوات غير المتاحة بعد
- [ ] hover tooltip لكل قناة باسمها
- [ ] data مصدرها config لا hardcoded في JSX

**الناتج:** Channels Section

---

### S1-T-003 | How It Works Section
**المسؤول:** Frontend Dev + Designer
**الوقت التقديري:** 5 ساعات
**User Story:** US-L-003

- [ ] 4 خطوات مرقّمة:
  1. "سجّل حسابك"
  2. "ربط قناتك المفضلة"
  3. "المساعد يعمل تلقائياً"
  4. "راقب وتحكّم من لوحة التحكم"
- [ ] أيقونة/رسم لكل خطوة
- [ ] على موبايل: vertical timeline

**الناتج:** How It Works Section

---

### S1-T-004 | Features Section
**المسؤول:** Frontend Dev
**الوقت التقديري:** 4 ساعات
**User Story:** US-L-001

- [ ] 6 ميزات رئيسية (بطاقات)
- [ ] كل بطاقة: أيقونة + عنوان + جملة وصف
- [ ] Grid responsive: 1→2→3 columns
- [ ] لا نص أكثر من 2 سطر في الوصف

**الناتج:** Features Section

---

### S1-T-005 | Pricing Section
**المسؤول:** Frontend Dev
**الوقت التقديري:** 8 ساعات
**User Story:** US-L-004

- [ ] 3 خطط: Free / Pro / Team
- [ ] تبديل شهري/سنوي مع badge "وفّر 20%"
- [ ] كل خطة: السعر، القائمة، CTA
- [ ] highlight للخطة الأكثر شيوعاً (Pro)
- [ ] data الخطط من config محوري (`lib/pricing.ts`)
- [ ] على موبايل: خطة واحدة بالعرض مع swipe

**الناتج:** Pricing Section متكاملة

---

### S1-T-006 | Social Proof Section
**المسؤول:** Frontend Dev
**الوقت التقديري:** 3 ساعات
**User Story:** US-L-005

- [ ] 4-6 شهادات في carousel / grid
- [ ] كل شهادة: avatar placeholder + اسم وهمي + الدور + القناة المستخدمة + الاقتباس
- [ ] لا أسماء حقيقية أو أرقام حتى تكتمل المراجعة
- [ ] Carousel على موبايل، grid على desktop

**الناتج:** Testimonials Section

---

### S1-T-007 | Footer
**المسؤول:** Frontend Dev
**الوقت التقديري:** 3 ساعات

- [ ] روابط: About, Pricing, Docs, Privacy, Terms, Status
- [ ] أيقونات social (Twitter/X, GitHub)
- [ ] حقوق النشر مع السنة الحالية (dynamic)
- [ ] Language switcher (EN/AR) placeholder

**الناتج:** Footer كامل

---

### S1-T-008 | صفحة Login
**المسؤول:** Frontend Dev + Auth Dev
**الوقت التقديري:** 5 ساعات
**User Story:** US-L-006

- [ ] فورم: البريد + كلمة المرور
- [ ] "تذكّرني" checkbox
- [ ] رابط "نسيت كلمة المرور؟"
- [ ] رسائل خطأ واضحة (بريد خاطئ / كلمة مرور خاطئة)
- [ ] loading state أثناء الـ auth
- [ ] redirect بعد النجاح إلى `/dashboard`

**الناتج:** صفحة Login كاملة

---

### S1-T-009 | صفحة Signup
**المسؤول:** Frontend Dev + Auth Dev
**الوقت التقديري:** 6 ساعات
**User Story:** US-L-006

- [ ] فورم: الاسم + البريد + كلمة المرور + تأكيد كلمة المرور
- [ ] Zod validation: كلمة مرور > 8 أحرف، بريد صحيح
- [ ] Terms of Service checkbox مع رابط
- [ ] إرسال بريد تأكيد بعد التسجيل
- [ ] رسالة "تحقق من بريدك" بعد النجاح

**الناتج:** صفحة Signup + بريد تأكيد

---

### S1-T-010 | صفحة Reset Password
**المسؤول:** Frontend Dev
**الوقت التقديري:** 4 ساعات
**User Story:** US-L-007

- [ ] صفحة 1: إدخال البريد
- [ ] صفحة 2: "تحقق من بريدك" (بعد الإرسال)
- [ ] صفحة 3: إدخال كلمة المرور الجديدة (عبر token في URL)
- [ ] صفحة 4: "تم تغيير كلمة المرور" + redirect لـ login

**الناتج:** Reset Password flow كامل

---

### S1-T-011 | SEO + Performance
**المسؤول:** Frontend Lead
**الوقت التقديري:** 4 ساعات
**User Story:** US-L-009

- [ ] `metadata` لكل صفحة (title, description)
- [ ] Open Graph tags
- [ ] `sitemap.ts` (Next.js auto sitemap)
- [ ] `robots.ts`
- [ ] تحسين الصور (next/image + WebP)
- [ ] قياس Lighthouse وتحقيق > 90

**الناتج:** Landing Page محسّنة للـ SEO والأداء

---

### S1-T-012 | اختبارات Sprint 1
**المسؤول:** كل المطورين
**الوقت التقديري:** 4 ساعات

- [ ] اختبار وحدة: Pricing config، validation schemas
- [ ] اختبار integration: Signup flow، Login flow
- [ ] اختبار e2e (Playwright): Landing → Signup → تأكيد → Dashboard

**الناتج:** test coverage > 70% للكود الجديد

---

## نقاط Sprint 1

| المهمة | النقاط |
|--------|--------|
| S1-T-001 Hero | 4 |
| S1-T-002 Channels | 2 |
| S1-T-003 How It Works | 3 |
| S1-T-004 Features | 2 |
| S1-T-005 Pricing | 5 |
| S1-T-006 Social Proof | 2 |
| S1-T-007 Footer | 2 |
| S1-T-008 Login | 3 |
| S1-T-009 Signup | 4 |
| S1-T-010 Reset Password | 3 |
| S1-T-011 SEO + Perf | 3 |
| S1-T-012 Tests | 3 |
| **المجموع** | **36** |
