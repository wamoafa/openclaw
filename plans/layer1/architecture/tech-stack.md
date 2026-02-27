# Layer 1 — Tech Stack

## القرارات التقنية

### Framework الرئيسي
**Next.js 15 (App Router)**

| السبب | التفاصيل |
|-------|---------|
| SSR للـ Landing | سرعة + SEO ممتاز |
| CSR للـ Dashboard | تجربة تطبيق سريعة |
| API Routes | لا حاجة لـ backend مستقل في Layer 1 |
| TypeScript | مطلوب في المشروع |

### المكتبات الأساسية

```
Layer 1 Tech Stack
├── Framework
│   └── Next.js 15 (App Router + TypeScript)
│
├── UI
│   ├── Tailwind CSS v4       ← الأساس
│   ├── shadcn/ui             ← مكونات جاهزة
│   └── Lucide Icons          ← الأيقونات
│
├── State Management
│   ├── Zustand               ← global state (auth, user)
│   └── TanStack Query v5     ← server state + caching
│
├── Auth
│   └── NextAuth.js v5        ← JWT sessions
│
├── Forms
│   ├── React Hook Form       ← إدارة الفورم
│   └── Zod                   ← validation
│
├── Charts (Admin)
│   └── Recharts              ← إحصائيات بسيطة
│
├── Testing
│   ├── Vitest                ← unit tests
│   └── Playwright            ← e2e tests
│
└── Dev Tools
    ├── ESLint + Oxlint
    ├── Prettier / Oxfmt
    └── Storybook (مستقبل)
```

## قرارات مقصودة

### لماذا لا Remix؟
Next.js أكثر استخداماً في الفريق وله ecosystem أضخم لـ dashboard components.

### لماذا Zustand وليس Redux؟
الـ state بسيط — auth + user preferences. Zustand أخف وأسرع في الإعداد.

### لماذا TanStack Query؟
كل بيانات الداشبورد تأتي من Core API — نحتاج caching + refetching + optimistic updates.

### لماذا NextAuth v5؟
يدعم JWT مباشرة، سهل ربطه بـ Core API، ويدعم OAuth مستقبلاً (Google, GitHub).

## بنية المجلدات المقترحة

```
apps/web/                       ← (داخل monorepo)
├── app/
│   ├── (public)/               ← Landing pages (no auth)
│   │   ├── page.tsx            ← /
│   │   ├── pricing/page.tsx
│   │   ├── how-it-works/page.tsx
│   │   └── channels/page.tsx
│   ├── (auth)/                 ← Auth pages
│   │   ├── login/page.tsx
│   │   ├── signup/page.tsx
│   │   └── reset-password/page.tsx
│   ├── dashboard/              ← User Dashboard (protected)
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── instances/
│   │   ├── channels/
│   │   ├── usage/
│   │   ├── billing/
│   │   └── settings/
│   └── admin/                  ← Admin Panel (admin role)
│       ├── layout.tsx
│       ├── page.tsx
│       ├── users/
│       ├── instances/
│       ├── billing/
│       └── system/
├── components/
│   ├── ui/                     ← shadcn components
│   ├── landing/                ← Landing-specific
│   ├── dashboard/              ← Dashboard-specific
│   ├── admin/                  ← Admin-specific
│   └── shared/                 ← مشترك
├── lib/
│   ├── api.ts                  ← API client (Core API)
│   ├── auth.ts                 ← NextAuth config
│   └── utils.ts
├── hooks/
│   ├── use-instances.ts
│   ├── use-billing.ts
│   └── use-channels.ts
├── store/
│   └── user-store.ts           ← Zustand
└── types/
    └── api.ts                  ← shared types من Core API
```

## متطلبات الأداء

| المقياس | الهدف |
|---------|-------|
| LCP (Landing) | < 2.5s |
| TTI (Dashboard) | < 3.5s |
| Bundle size (initial) | < 200KB gzipped |
| Lighthouse Score | > 90 |

## البيئات

```
local     → http://localhost:3000
staging   → https://staging.openclaw.ai
production → https://openclaw.ai (أو app.openclaw.ai)
```
