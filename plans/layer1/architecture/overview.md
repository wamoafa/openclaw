# Layer 1 — Architecture Overview

## ما هو Layer 1؟

Layer 1 هو الواجهة الكاملة التي يتعامل معها المستخدم والمسؤول.
يتكون من ثلاثة أسطح (surfaces):

```
┌─────────────────────────────────────────────────────┐
│                    LAYER 1                          │
│                                                     │
│  ┌─────────────────┐  ┌───────────────────────────┐ │
│  │  Landing Page   │  │     Dashboard App         │ │
│  │  (تسويق + CTA)  │  │                           │ │
│  │                 │  │  ┌───────────┐ ┌────────┐  │ │
│  │  - Hero         │  │  │   User    │ │ Admin  │  │ │
│  │  - Features     │  │  │  Panel    │ │ Panel  │  │ │
│  │  - Pricing      │  │  └───────────┘ └────────┘  │ │
│  │  - How it works │  │                           │ │
│  │  - Signup CTA   │  └───────────────────────────┘ │
│  └─────────────────┘                               │
└──────────────────────────┬──────────────────────────┘
                           │ REST/WebSocket
                    Layer 2 (Core API)
```

## مبادئ التصميم

| المبدأ | التطبيق |
|--------|---------|
| **لا تعرف الخلفية** | Dashboard لا يكلّم VPS مباشرة — كل شيء عبر Core API |
| **مناسب لغير التقنيين** | لغة بسيطة، خطوات واضحة، لا مصطلحات تقنية |
| **Mobile-first** | تصميم يبدأ من الشاشة الصغيرة |
| **أداء سريع** | Server-side rendering للصفحات العامة، CSR للداشبورد |
| **قابل للتوسع** | كل جزء مستقل — تقدر تستبدل Landing دون لمس Dashboard |

## الحالات (States) للمستخدم

```
زائر (Visitor)
    │
    ▼ يسجّل
مستخدم مجرّب (Trial)
    │
    ▼ يدفع
مستخدم نشط (Active)
    │
    ▼ ترقية
مستخدم Plus
```

## ما يراه كل نوع

| السطح | الزائر | Trial | Active | Admin |
|-------|--------|-------|--------|-------|
| Landing Page | ✅ كامل | ✅ | ✅ | ✅ |
| User Dashboard | ❌ | ✅ محدود | ✅ كامل | ✅ |
| Admin Panel | ❌ | ❌ | ❌ | ✅ |

## الـ Routes الرئيسية

### Landing Page (Public)
```
/                   → الصفحة الرئيسية
/pricing            → التسعير
/how-it-works       → كيف يعمل
/channels           → القنوات المدعومة
/blog               → المدونة (مستقبل)
/login              → تسجيل الدخول
/signup             → إنشاء حساب
/verify-email       → تأكيد البريد
/reset-password     → استعادة كلمة المرور
```

### User Dashboard (Protected)
```
/dashboard                  → الرئيسية (overview)
/dashboard/instances        → VPS instances
/dashboard/instances/new    → إنشاء instance جديد
/dashboard/instances/:id    → تفاصيل instance
/dashboard/channels         → ربط القنوات
/dashboard/usage            → الاستخدام والإحصائيات
/dashboard/billing          → الفواتير والاشتراك
/dashboard/settings         → الإعدادات
/dashboard/support          → الدعم
```

### Admin Panel (Admin Only)
```
/admin                      → لوحة المسؤول
/admin/users                → إدارة المستخدمين
/admin/users/:id            → تفاصيل مستخدم
/admin/instances            → كل الـ instances
/admin/billing              → إيرادات وفواتير
/admin/system               → صحة النظام
/admin/announcements        → الإعلانات
/admin/settings             → إعدادات النظام
```
