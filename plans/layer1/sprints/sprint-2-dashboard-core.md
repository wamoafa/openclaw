# Sprint 2 — Dashboard Core (Onboarding + Instances)

**المدة:** أسبوعان (10 أيام)
**الهدف:** Onboarding flow كامل، Instance management، ربط مع Core API
**User Stories المغطاة:** US-D-001 → US-D-006

---

## المتطلبات المسبقة

- Core API endpoint لـ Instances متاح (حتى لو staging)
- Auth tokens تعمل بين Layer 1 و Layer 2
- Schema أنواع الـ API محدد في `types/api.ts`

---

## متى ينتهي Sprint 2 بنجاح؟

- [ ] مستخدم جديد يكمل Onboarding ويصل لـ Dashboard
- [ ] يستطيع إنشاء Instance ورؤية حالته
- [ ] TanStack Query يجلب البيانات من Core API
- [ ] الأخطاء تظهر بطريقة مفيدة (لا errors خام)

---

## المهام

### S2-T-001 | API Client
**المسؤول:** Frontend Lead
**الوقت التقديري:** 4 ساعات

- [ ] إنشاء `lib/api.ts`: wrapper حول fetch مع:
  - Base URL من env
  - Auto-attach JWT token من NextAuth session
  - Error handling موحّد (ApiError class)
  - TypeScript generics: `api.get<T>()`, `api.post<T>()`
- [ ] `hooks/use-api.ts`: integration مع TanStack Query

**الناتج:** API client جاهز

---

### S2-T-002 | Types من Core API
**المسؤول:** Frontend Lead + Backend
**الوقت التقديري:** 3 ساعات

- [ ] تعريف `types/api.ts`:
  ```typescript
  type Instance = { id, name, status, region, channels, createdAt }
  type Channel = { id, type, status, connectedAt }
  type User = { id, email, name, plan, usage }
  type Plan = { id, name, price, limits }
  ```
- [ ] مشاركة Types مع Core API team (أو توليدها من OpenAPI spec)

**الناتج:** types دقيقة لكل الـ API responses

---

### S2-T-003 | Onboarding Flow
**المسؤول:** Frontend Dev
**الوقت التقديري:** 8 ساعات
**User Story:** US-D-001

- [ ] Component `Onboarding` مع stepper:
  - الخطوة 1: "مرحباً! دعنا نعرّفك على OpenClaw" (welcome screen)
  - الخطوة 2: إنشاء أول Instance (يستدعي S2-T-004)
  - الخطوة 3: ربط أول قناة (يستدعي Sprint 3)
  - الخطوة 4: الاختبار + الإكمال
- [ ] حفظ تقدم الـ onboarding في user record
- [ ] لو المستخدم أغلق في المنتصف: يكمل من حيث توقف
- [ ] Celebration animation عند الاكتمال

**الناتج:** Onboarding كامل

---

### S2-T-004 | Dashboard Home (Overview)
**المسؤول:** Frontend Dev
**الوقت التقديري:** 6 ساعات
**User Story:** US-D-002

- [ ] بطاقات الـ Instances مع real-time status
- [ ] "لا instances بعد" empty state مع CTA
- [ ] تنبيهات فعّالة (قناة مفصولة، instance متوقف)
- [ ] عداد رسائل اليوم
- [ ] Skeleton loading أثناء الجلب

**الناتج:** Dashboard Home

---

### S2-T-005 | قائمة Instances
**المسؤول:** Frontend Dev
**الوقت التقديري:** 5 ساعات
**User Story:** US-D-003

- [ ] `hooks/use-instances.ts`: TanStack Query hook
- [ ] جدول / بطاقات الـ instances
- [ ] فلتر الحالة (نشط/موقوف)
- [ ] Pagination
- [ ] زر "إنشاء Instance" يفتح Modal أو ينقل لصفحة جديدة

**الناتج:** صفحة قائمة Instances

---

### S2-T-006 | إنشاء Instance
**المسؤول:** Frontend Dev
**الوقت التقديري:** 8 ساعات
**User Story:** US-D-004

- [ ] Form multi-step:
  - الخطوة 1: اسم الـ Instance
  - الخطوة 2: اختيار المنطقة مع مؤشر Latency
  - الخطوة 3: مراجعة + تأكيد
- [ ] Progress bar أثناء الإنشاء (polling كل 3 ثوانٍ)
- [ ] رسالة نجاح + redirect لصفحة Instance
- [ ] رسالة خطأ واضحة لو فشل الإنشاء
- [ ] حد الخطة: رسالة "ترقّ لإنشاء المزيد"

**الناتج:** Create Instance flow

---

### S2-T-007 | تفاصيل Instance
**المسؤول:** Frontend Dev
**الوقت التقديري:** 7 ساعات
**User Story:** US-D-005

- [ ] Header: الاسم + الحالة + أزرار التحكم
- [ ] Tab: "القنوات" - قائمة القنوات المرتبطة
- [ ] Tab: "Logs" - آخر 50 سطر مع تحديث
- [ ] Tab: "الإعدادات" - نموذج AI، timeout, etc.
- [ ] أزرار: إيقاف مؤقت / إعادة تشغيل / حذف
- [ ] WebSocket لتحديث الحالة لحظياً

**الناتج:** صفحة Instance Details

---

### S2-T-008 | حذف Instance
**المسؤول:** Frontend Dev
**الوقت التقديري:** 3 ساعات
**User Story:** US-D-006

- [ ] Dialog تأكيد مع كتابة الاسم
- [ ] Optimistic update (يختفي من القائمة فوراً)
- [ ] Rollback لو API رجع error
- [ ] Redirect لقائمة Instances بعد النجاح

**الناتج:** Delete Instance

---

### S2-T-009 | Error States + Loading States
**المسؤول:** Frontend Dev
**الوقت التقديري:** 4 ساعات

- [ ] Skeleton loaders لكل صفحة (Instance list, Instance detail)
- [ ] Error boundary component مع "حاول مرة ثانية"
- [ ] Empty states لكل قائمة فارغة (مع CTA واضح)
- [ ] Toast notifications (Sonner أو shadcn toast):
  - ✅ نجاح الإنشاء/الحذف
  - ❌ فشل أي عملية

**الناتج:** UX متسق لكل الحالات

---

### S2-T-010 | اختبارات Sprint 2
**المسؤول:** كل المطورين
**الوقت التقديري:** 4 ساعات

- [ ] unit: API client error handling
- [ ] unit: Instance form validation
- [ ] integration: Create Instance flow (mocked API)
- [ ] integration: Onboarding step progression

**الناتج:** tests تغطي منطق Sprint 2

---

## نقاط Sprint 2

| المهمة | النقاط |
|--------|--------|
| S2-T-001 API Client | 3 |
| S2-T-002 Types | 2 |
| S2-T-003 Onboarding | 5 |
| S2-T-004 Dashboard Home | 4 |
| S2-T-005 Instance List | 3 |
| S2-T-006 Create Instance | 5 |
| S2-T-007 Instance Detail | 5 |
| S2-T-008 Delete Instance | 2 |
| S2-T-009 Error/Loading | 3 |
| S2-T-010 Tests | 3 |
| **المجموع** | **35** |
