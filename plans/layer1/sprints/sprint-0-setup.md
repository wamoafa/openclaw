# Sprint 0 — Foundation Setup

**المدة:** أسبوع واحد (5 أيام)
**الهدف:** بيئة تطوير جاهزة، CI/CD، design system أساسي، auth skeleton

---

## متى ينتهي Sprint 0 بنجاح؟

- [ ] مطور جديد يشغّل المشروع بأمر واحد (`pnpm install && pnpm dev`)
- [ ] الصفحة الرئيسية تظهر في المتصفح بدون أخطاء
- [ ] CI يمرر: build + lint + typecheck + tests
- [ ] صفحة login تعمل (حتى لو mock)
- [ ] Design tokens (ألوان، fonts) محددة ومطبّقة

---

## المهام

### S0-T-001 | إعداد Next.js Project
**المسؤول:** Frontend Lead
**الوقت التقديري:** 4 ساعات

- [ ] `npx create-next-app@latest apps/web --typescript --tailwind --app --src-dir`
- [ ] إضافة `apps/web` لـ pnpm workspace
- [ ] إعداد `tsconfig.json` بـ strict mode
- [ ] إضافة path aliases: `@/components`, `@/lib`, `@/hooks`, `@/store`
- [ ] إعداد `.env.example` مع كل المتغيرات المطلوبة

**التبعيات:** لا شيء
**الناتج:** مشروع Next.js يشتغل على localhost:3000

---

### S0-T-002 | إعداد Tailwind + shadcn/ui
**المسؤول:** Frontend Lead
**الوقت التقديري:** 3 ساعات

- [ ] تهيئة Tailwind v4
- [ ] تعريف CSS variables للـ design tokens:
  ```css
  --color-primary, --color-secondary, --color-destructive
  --font-sans, --font-mono
  --radius-sm, --radius-md, --radius-lg
  ```
- [ ] تثبيت shadcn/ui: `npx shadcn@latest init`
- [ ] إضافة المكونات الأساسية: Button, Input, Card, Dialog, Badge, Table, Tabs

**التبعيات:** S0-T-001
**الناتج:** design system يمكن استخدامه في أي component

---

### S0-T-003 | إعداد NextAuth.js
**المسؤول:** Backend/Auth Dev
**الوقت التقديري:** 4 ساعات

- [ ] تثبيت `next-auth@beta` (v5)
- [ ] تهيئة `auth.config.ts` مع Credentials provider
- [ ] إعداد JWT strategy (لا database sessions في البداية)
- [ ] Middleware لحماية routes الـ dashboard والـ admin
- [ ] صفحة login أساسية (بدون تصميم نهائي)
- [ ] صفحة signup أساسية
- [ ] دالة `signIn` و `signOut` تعمل

**التبعيات:** S0-T-001
**الناتج:** auth يعمل (حتى لو credentials hardcoded مؤقتاً)

---

### S0-T-004 | إعداد CI/CD
**المسؤول:** DevOps / أي مطور
**الوقت التقديري:** 3 ساعات

- [ ] GitHub Actions workflow لـ `apps/web`:
  - `pnpm install`
  - `pnpm build`
  - `pnpm lint` / `pnpm check`
  - `pnpm tsgo`
  - `pnpm test`
- [ ] تحديد `.github/labeler.yml` لإضافة label `web` على PRs
- [ ] إعداد Vercel Preview للـ PRs (أو Netlify)

**التبعيات:** S0-T-001
**الناتج:** CI يمرر على كل PR

---

### S0-T-005 | إعداد Zustand + TanStack Query
**المسؤول:** Frontend Dev
**الوقت التقديري:** 2 ساعة

- [ ] تثبيت `zustand` + `@tanstack/react-query`
- [ ] إنشاء `store/user-store.ts` (user state, auth state)
- [ ] إنشاء `lib/query-client.ts` مع default options
- [ ] `Providers` component يلف التطبيق
- [ ] DevTools للـ Query في development mode

**التبعيات:** S0-T-001
**الناتج:** state management جاهز للاستخدام

---

### S0-T-006 | Layout أساسي + Navigation Shell
**المسؤول:** Frontend Dev
**الوقت التقديري:** 4 ساعات

- [ ] `(public)/layout.tsx`: Header + Footer للصفحات العامة
- [ ] `dashboard/layout.tsx`: Sidebar + Top bar للداشبورد
- [ ] `admin/layout.tsx`: مشابه للداشبورد مع تمييز بصري
- [ ] Navigation items قابلة للتهيئة (مش hardcoded)
- [ ] Responsive sidebar (collapse على موبايل)

**التبعيات:** S0-T-002, S0-T-003
**الناتج:** shell جاهز لإضافة المحتوى في Sprints قادمة

---

### S0-T-008 | إعداد PostgreSQL + Drizzle ORM
**المسؤول:** Backend Dev
**الوقت التقديري:** 3 ساعات

- [ ] تثبيت `drizzle-orm`, `drizzle-kit`, `postgres` (driver)
- [ ] إنشاء `lib/db.ts` — اتصال الـ DB عبر `postgres.js`
- [ ] إنشاء `db/schema.ts` — تعريف الجداول الأولية:
  - `users` (id, email, password_hash, role, created_at)
  - `sessions` (اختياري إذا استخدمنا DB sessions مع NextAuth)
- [ ] إعداد `drizzle.config.ts` للـ migrations
- [ ] تشغيل أول migration: `bunx drizzle-kit migrate`
- [ ] إضافة `DATABASE_URL` لـ `.env.example`
- [ ] التحقق من الاتصال بـ PostgreSQL على `45.55.253.17` في البيئة المحلية

**التبعيات:** S0-T-001
**الناتج:** قاعدة بيانات جاهزة + schema مُعرَّف + ORM يعمل

---

### S0-T-007 | إعداد Vitest + أول اختبار
**المسؤول:** أي مطور
**الوقت التقديري:** 2 ساعة

- [ ] تهيئة vitest لـ `apps/web`
- [ ] اختبار أول component (Button أو utility function)
- [ ] اختبار التحقق من الأدوار (admin vs user middleware)
- [ ] إعداد coverage threshold أولية

**التبعيات:** S0-T-001
**الناتج:** test suite تشتغل مع CI

---

## ما خارج Sprint 0

- ❌ تصميم Landing Page النهائي
- ❌ ربط مع Core API (لا يزال قيد البناء)
- ❌ Stripe integration
- ❌ صفحات محتوى حقيقية

---

## نقاط Sprint 0

| المهمة | النقاط |
|--------|--------|
| S0-T-001 | 3 |
| S0-T-002 | 2 |
| S0-T-003 | 3 |
| S0-T-004 | 2 |
| S0-T-005 | 1 |
| S0-T-006 | 3 |
| S0-T-007 | 1 |
| S0-T-008 (DB) | 2 |
| **المجموع** | **17** |
