# Layer 1 — خطة التخطيط الكاملة

## ما هو هذا المجلد؟

خطة تفصيلية لبناء **Layer 1** — واجهة المستخدم الكاملة لمنتج OpenClaw.
تشمل: Landing Page، User Dashboard، Admin Panel.

---

## هيكل الملفات

```
plans/layer1/
├── README.md                          ← أنت هنا
│
├── architecture/
│   ├── overview.md                    ← البنية العامة + الـ Routes
│   └── tech-stack.md                  ← التقنيات + هيكل المجلدات
│
├── user-stories/
│   ├── 01-landing-page.md             ← 9 قصص مستخدم للـ Landing
│   ├── 02-user-dashboard.md           ← 14 قصة مستخدم للداشبورد
│   └── 03-admin-panel.md              ← 12 قصة مستخدم للـ Admin
│
├── sprints/
│   ├── sprint-0-setup.md              ← أسبوع 1: بيئة تطوير + Auth Shell
│   ├── sprint-1-landing.md            ← أسبوعان: Landing Page كاملة
│   ├── sprint-2-dashboard-core.md     ← أسبوعان: Onboarding + Instances
│   ├── sprint-3-channels-billing.md   ← أسبوعان: Channels + Billing
│   ├── sprint-4-admin.md              ← أسبوعان: Admin Panel
│   └── sprint-5-polish.md             ← أسبوع: Polish + QA + i18n
│
├── tasks/
│   ├── backlog.md                     ← Backlog كامل مرتّب بالأولوية
│   └── definition-of-done.md         ← معايير الاكتمال + Story Points
│
└── design/
    └── wireframes.md                  ← ASCII wireframes للصفحات الرئيسية
```

---

## الخلاصة التنفيذية

### المدة الإجمالية: ~11 أسبوع

| Sprint | المدة | الهدف | النقاط |
|--------|-------|-------|--------|
| Sprint 0 | أسبوع | Setup + CI + Auth | 15 |
| Sprint 1 | أسبوعان | Landing Page كاملة | 36 |
| Sprint 2 | أسبوعان | Dashboard + Instances | 35 |
| Sprint 3 | أسبوعان | Channels + Billing | 35 |
| Sprint 4 | أسبوعان | Admin Panel | 43 |
| Sprint 5 | أسبوع | Polish + QA + i18n | 28 |
| **الإجمالي** | **~11 أسبوع** | | **192 نقطة** |

---

## التقنيات المختارة

```
Next.js 15 (App Router)     ← Framework
TypeScript                  ← اللغة
Tailwind CSS v4             ← التصميم
shadcn/ui                   ← مكونات UI
NextAuth.js v5              ← Auth
PostgreSQL 16               ← قاعدة البيانات ✅ مُحدَّد
Drizzle ORM                 ← Database ORM (TypeScript-first)
Zustand                     ← Global State
TanStack Query v5           ← Server State
Zod + React Hook Form       ← Forms
Recharts                    ← Charts (Admin)
Stripe                      ← Billing
Vitest + Playwright         ← Testing
DigitalOcean                ← Hosting ✅ مُحدَّد (45.55.253.17)
```

---

## الـ Surfaces الثلاثة

### 1. Landing Page (Public)
- **الهدف:** تحويل الزوار إلى مسجّلين
- **الأولوية:** P0 — أول ما يُبنى
- **Sprint:** S1

### 2. User Dashboard (Protected)
- **الهدف:** تمكين المستخدم من إدارة مساعده بسهولة
- **الأقسام:** Overview، Instances، Channels، Usage، Billing، Settings
- **Sprints:** S2, S3

### 3. Admin Panel (Admin Role)
- **الهدف:** تمكين الفريق من إدارة المنتج والمستخدمين
- **الأقسام:** Overview، Users، Instances، Revenue، System، Audit
- **Sprint:** S4

---

## التبعيات الخارجية

| التبعية | Sprint الذي يحتاجها | ملاحظة |
|---------|---------------------|--------|
| Core API (Layer 2) | S2+ | يجب أن تكون endpoints Instance جاهزة |
| Stripe Account | S3 | لـ Billing integration |
| Email Service (Resend/SES) | S1 | لـ verification emails |
| Hosting (DigitalOcean) | S0 | Droplet جاهز: `45.55.253.17` ✅ |

---

## كيف تستخدم هذا التخطيط؟

### لمدير المشروع:
1. ابدأ بـ `tasks/backlog.md` لترتيب الأولويات
2. كل Sprint له ملف مفصّل في `sprints/`
3. كل مهمة لها وقت تقديري + نقاط Story
4. راجع `tasks/definition-of-done.md` قبل قبول أي مهمة

### للمطور:
1. ابدأ بـ `architecture/tech-stack.md` لفهم البنية
2. ارجع لـ `user-stories/` لتفهم السياق قبل البناء
3. كل مهمة في الـ Sprints لها Acceptance Criteria واضحة

### للمصمم:
1. `design/wireframes.md` نقطة البداية
2. راجع `user-stories/` لتفهم احتياجات كل شخصية
3. كل User Story مكتوب من منظور المستخدم لا التقنية

---

## القرارات المعلّقة

راجع `tasks/backlog.md` قسم "الأسئلة المعلّقة" — هناك قرارات
تحتاج موافقة قبل بدء S1 وS3:

- سعر الخطط
- هل يوجد Free plan
- الـ payment processor
- OAuth من Sprint 1 أو 2
- هل Admin في subdomain منفصل؟
