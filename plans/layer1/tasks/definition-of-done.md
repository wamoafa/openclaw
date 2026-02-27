# Definition of Done — Layer 1

تُعتبر أي مهمة أو User Story "منتهية" فقط عند استيفاء **كل** البنود التالية.

---

## مستوى الكود

- [ ] الكود يمرر `pnpm build` بدون أخطاء
- [ ] الكود يمرر `pnpm check` (Oxlint) بدون warnings جديدة
- [ ] الكود يمرر `pnpm tsgo` (type check) بدون أخطاء
- [ ] لا `any` صريحة — استخدم types دقيقة
- [ ] لا `console.log` في production code
- [ ] لا أسرار أو بيانات حساسة hardcoded

## مستوى الاختبار

- [ ] اختبار وحدة لكل logic جديدة (utils, hooks, validators)
- [ ] اختبار integration لكل form flow أو API interaction
- [ ] `pnpm test` يمرر بالكامل (لا failing tests)
- [ ] Coverage لا تنزل عن 70% للملفات المعدّلة

## مستوى UX/UI

- [ ] الصفحة/المكوّن يعمل على: 375px (mobile) + 768px (tablet) + 1440px (desktop)
- [ ] حالة Loading معروضة أثناء الجلب (Skeleton أو Spinner)
- [ ] حالة Error تعرض رسالة مفيدة وليس stack trace
- [ ] حالة Empty State واضحة (لو القائمة فارغة)
- [ ] لا text overflow أو layout breaks واضحة

## مستوى Accessibility

- [ ] كل Form inputs لها labels مرتبطة (`<label htmlFor>`)
- [ ] الأزرار التفاعلية لها `aria-label` إذا لم يكن فيها نص
- [ ] focus visible على كل العناصر التفاعلية
- [ ] Color contrast لا يقل عن WCAG AA (4.5:1)

## مستوى Performance (للصفحات الكاملة فقط)

- [ ] لا images بدون `width` و `height` (CLS = 0)
- [ ] Fonts تُحمّل بـ `font-display: swap`
- [ ] Dynamic imports للمكونات الثقيلة (Charts, Rich Editors)

## مستوى الـ PR

- [ ] PR title واضح يصف التغيير (مثل `feat(dashboard): add instance delete confirmation`)
- [ ] وصف PR يشير للـ User Story (مثل `Closes US-D-006`)
- [ ] Screenshots أو video للتغييرات البصرية
- [ ] Review من مطور آخر على الأقل
- [ ] CI يمرر (build + lint + types + tests)
- [ ] لا merge conflicts

## مستوى الأمان

- [ ] User inputs مُعقّمة (Zod validation على الـ server)
- [ ] لا XSS: لا `dangerouslySetInnerHTML` بدون sanitization
- [ ] الـ admin routes محمية server-side (لا client-only guards فقط)
- [ ] لا بيانات حساسة في URL parameters

---

## Acceptance Criteria Template

كل User Story يجب أن يكتب بهذا الشكل:

```
**بوصفي** [الشخصية]،
**أريد** [الفعل]،
**حتى** [الفائدة].

**معايير القبول:**
- [ ] ...
- [ ] ...

**الحجم:** XS / S / M / L / XL
**الأولوية:** P0 / P1 / P2 / P3
```

---

## مقياس الحجم (Story Points)

| الحجم | النقاط | التقدير |
|-------|--------|---------|
| XS | 1 | < 2 ساعات |
| S | 2 | 2-4 ساعات |
| M | 3 | 4-8 ساعات |
| L | 5 | 1-2 يوم |
| XL | 8 | 2-3 أيام |
| XXL | 13 | > 3 أيام (يجب التقسيم) |

> مهمة بحجم XXL يجب تقسيمها قبل وضعها في Sprint.

---

## Sprint Review Checklist

في نهاية كل Sprint:
- [ ] Demo للـ Stakeholders على staging
- [ ] كل User Stories P0 في الـ Sprint مكتملة
- [ ] Velocity محسوبة (نقاط مكتملة / إجمالي النقاط)
- [ ] Retrospective: ما نجح، ما لم ينجح، ماذا نغير
- [ ] Backlog مُحدَّث بأي مهام اكتُشفت
- [ ] Sprint التالي مخطّط ومُقدَّر
