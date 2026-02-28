# Layer 2 — Backlog

## الأولويات

### P0 — حرج (يمنع الإطلاق)

| ID | المهمة | Sprint | النقاط |
|----|--------|--------|--------|
| L2-001 | Auth — register + login + JWT | S0 | 5 |
| L2-002 | PostgreSQL + Drizzle + migrations | S0 | 3 |
| L2-003 | Instance CRUD + lifecycle | S1 | 5 |
| L2-004 | Node SSH controller | S1 | 5 |
| L2-005 | Gateway WebSocket RPC client | S1 | 5 |
| L2-006 | Channel linking (WhatsApp QR) | S2 | 5 |
| L2-007 | Stripe subscription flow | S3 | 5 |
| L2-008 | Plan limits enforcement | S3 | 3 |
| L2-009 | Deploy to `45.55.253.17` | S1 | 2 |

### P1 — مهم (مطلوب قبل الإطلاق)

| ID | المهمة | Sprint | النقاط |
|----|--------|--------|--------|
| L2-010 | Channel linking (Telegram, Discord) | S2 | 3 |
| L2-011 | Usage tracking + limits | S3 | 5 |
| L2-012 | Stripe webhooks | S3 | 3 |
| L2-013 | Admin — user management | S4 | 5 |
| L2-014 | Admin — system health | S4 | 5 |
| L2-015 | WebSocket events (real-time) | S1 | 3 |
| L2-016 | BullMQ — provisioning queue | S1 | 5 |
| L2-017 | Email notifications | S4 | 3 |
| L2-018 | Audit logging | S3 | 2 |

### P2 — مرغوب (يحسّن التجربة)

| ID | المهمة | Sprint | النقاط |
|----|--------|--------|--------|
| L2-019 | Agent config management API | S2 | 3 |
| L2-020 | Session management API | S2 | 3 |
| L2-021 | Admin — revenue dashboard | S4 | 3 |
| L2-022 | Admin — audit logs API | S4 | 2 |
| L2-023 | Admin — announcements | S4 | 2 |
| L2-024 | Instance logs endpoint | S1 | 2 |
| L2-025 | Rate limiting | S4 | 2 |
| L2-026 | OpenAPI documentation | S4 | 2 |

### P3 — مستقبلي (بعد الإطلاق)

| ID | المهمة | Sprint | النقاط |
|----|--------|--------|--------|
| L2-027 | DigitalOcean API — auto provisioning | — | 8 |
| L2-028 | Node auto-scaling | — | 13 |
| L2-029 | Multi-region support | — | 8 |
| L2-030 | OAuth login (Google, GitHub) | — | 5 |
| L2-031 | Two-factor authentication (2FA) | — | 5 |
| L2-032 | API keys for external access | — | 5 |
| L2-033 | Slack/Discord notifications for admin | — | 3 |
| L2-034 | Prometheus metrics export | — | 3 |
| L2-035 | Backup + restore for instances | — | 8 |

---

## القرارات المُحسومة ✅

| القرار | الاختيار | التاريخ |
|--------|---------|---------|
| قاعدة البيانات | PostgreSQL 16 | 2026-02-28 |
| ORM | Drizzle ORM | 2026-02-28 |
| الاستضافة | DigitalOcean | 2026-02-28 |
| IP الخادم | `45.55.253.17` | 2026-02-28 |
| API Framework | Hono | 2026-02-28 |
| Job Queue | BullMQ + Redis | 2026-02-28 |
| Auth | JWT (jose) | 2026-02-28 |
| التحكم بـ openclaw | WebSocket Gateway RPC + SSH | 2026-02-28 |

## القرارات المعلّقة

| السؤال | الخيارات | يؤثر على |
|--------|---------|---------|
| كل مستخدم VPS مستقل أو مشترك؟ | dedicated / shared node | S1 — node allocation |
| عدد instances لكل node؟ | 1-5 / 5-20 / unlimited | S1 — scaling |
| هل نستخدم DigitalOcean API للـ provisioning أو nodes جاهزة؟ | API / pre-provisioned | S1 — MVP يدوي، P3 تلقائي |
| أسعار الخطط؟ | TBD | S3 — seed data |
| حدود كل خطة (messages, instances, channels)؟ | TBD | S3 |
| هل الـ API والداشبورد على نفس الـ domain أو subdomain؟ | `api.openclaw.ai` / `openclaw.ai/api` | S0 — CORS + deploy |

---

## ملخص النقاط

| Sprint | النقاط |
|--------|--------|
| Sprint 0 — Setup | 18 |
| Sprint 1 — Users + Instances | 35 |
| Sprint 2 — Channels | 27 |
| Sprint 3 — Billing + Usage | 25 |
| Sprint 4 — Admin + Monitoring | 30 |
| **الإجمالي** | **135 نقطة** |

---

## التبعيات بين Layer 1 و Layer 2

```
Layer 1 Sprint 0 (Setup)         ← لا يحتاج Layer 2
Layer 1 Sprint 1 (Landing)       ← لا يحتاج Layer 2
                                    ↑
Layer 2 Sprint 0 (Setup)         → Auth endpoints جاهزة
Layer 2 Sprint 1 (Instances)     → Instance + Node API جاهز
                                    ↑
Layer 1 Sprint 2 (Dashboard)     ← يحتاج Layer 2 S0+S1 ✅
Layer 2 Sprint 2 (Channels)      → Channel API جاهز
                                    ↑
Layer 1 Sprint 3 (Billing)       ← يحتاج Layer 2 S2+S3 ✅
Layer 2 Sprint 3 (Billing)       → Billing + Usage API جاهز
                                    ↑
Layer 1 Sprint 4 (Admin)         ← يحتاج Layer 2 S4 ✅
Layer 2 Sprint 4 (Admin)         → Admin API جاهز
```

**التوصية:** Layer 2 يسبق Layer 1 بـ sprint واحد على الأقل.
