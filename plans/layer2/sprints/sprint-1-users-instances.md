# Sprint 1 — Users + Instances + Node Control

**المدة:** أسبوعان (10 أيام)
**الهدف:** المستخدم يقدر ينشئ instance ويشغّله ويوقفه — مع تحكم فعلي بالسيرفر

---

## متى ينتهي Sprint 1 بنجاح؟

- [ ] `POST /instances` → ينشئ instance ويشغّل openclaw gateway على الـ node
- [ ] `GET /instances/:id/status` → يرجع حالة مباشرة من الـ gateway (WebSocket RPC)
- [ ] `POST /instances/:id/stop` → يوقف الـ gateway على الـ node
- [ ] `POST /instances/:id/restart` → يعيد تشغيل الـ gateway
- [ ] Node pool جاهز — على الأقل node واحد (`45.55.253.17`)
- [ ] WebSocket events تُرسل عند تغيّر حالة instance
- [ ] BullMQ يعالج مهام provisioning

---

## المهام

### S1-T-001 | Node Service — SSH Controller
**المسؤول:** Backend Dev (Systems)
**الوقت:** 8 ساعات | **النقاط:** 5

- [ ] `src/services/node.service.ts`:
  - `connectSSH(nodeId)` → إنشاء اتصال SSH عبر node-ssh
  - `installOpenClaw(nodeId)` → `npm i -g openclaw@latest`
  - `startGateway(nodeId, config)` → start مع token + port
  - `stopGateway(nodeId)` → `pkill -f openclaw-gateway`
  - `restartGateway(nodeId)` → stop + start
  - `getSystemResources(nodeId)` → CPU, RAM, Disk
  - `isGatewayRunning(nodeId)` → process check
- [ ] `src/lib/ssh.ts` — SSH connection pool (reuse connections)
- [ ] SSH key management — private key path from config
- [ ] Command whitelist — لا يُنفَّذ أمر عشوائي
- [ ] Timeout + error handling لكل SSH command

**التبعيات:** S0
**الناتج:** نقدر نتحكم بأي node عبر SSH

---

### S1-T-002 | Gateway Client — WebSocket RPC
**المسؤول:** Backend Dev
**الوقت:** 8 ساعات | **النقاط:** 5

- [ ] `src/services/gateway-client.ts`:
  - `connect(nodeIp, port, token)` → WebSocket اتصال
  - `call(method, params)` → RPC call (promise-based)
  - `subscribe(event, callback)` → listen to events
  - `disconnect()`
- [ ] Connection pool — اتصال واحد per node (reconnect تلقائي)
- [ ] Methods مستخدمة:
  - `health` → فحص الصحة
  - `status` → حالة الـ gateway
  - `channels.status` → حالة القنوات
  - `config.get` / `config.set` / `config.apply` → تعديل الإعدادات
  - `sessions.list` → الجلسات النشطة
  - `send` → إرسال رسالة
- [ ] Error handling + reconnection strategy
- [ ] Event forwarding إلى Socket.io (للداشبورد)

**التبعيات:** S0, node مع openclaw gateway يعمل
**الناتج:** نتكلم مع openclaw gateway عبر WebSocket RPC

---

### S1-T-003 | Node CRUD + Registration (Admin)
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] `src/routes/admin.ts` — node management:
  - `POST /admin/nodes` → تسجيل node جديد (IP + SSH key)
  - `GET /admin/nodes` → قائمة كل الـ nodes
  - `GET /admin/nodes/:id` → تفاصيل + health
  - `PATCH /admin/nodes/:id` → تعديل
  - `DELETE /admin/nodes/:id` → حذف (مع إيقاف instances)
  - `POST /admin/nodes/:id/health-check` → فحص فوري
- [ ] Seed node: إضافة `45.55.253.17` كـ node أولي

**التبعيات:** S1-T-001
**الناتج:** Admin يقدر يضيف/يحذف nodes

---

### S1-T-004 | Instance CRUD
**المسؤول:** Backend Dev
**الوقت:** 6 ساعات | **النقاط:** 5

- [ ] `src/services/instance.service.ts`:
  - `create(userId, name)` → اختيار node + إنشاء + queue provisioning
  - `list(userId)` → كل instances المستخدم
  - `getById(id, userId)` → تفاصيل مع حالة مباشرة
  - `update(id, data)` → تعديل الاسم/الإعدادات
  - `delete(id)` → إيقاف + حذف
  - `start(id)` → تشغيل gateway
  - `stop(id)` → إيقاف gateway
  - `restart(id)` → إعادة تشغيل
  - `getStatus(id)` → حالة مباشرة عبر Gateway RPC
- [ ] `src/routes/instances.ts` — كل الـ endpoints
- [ ] Node selection strategy: اختيار الـ node الأقل حملاً
- [ ] Plan limit check: عدد instances المسموح

**التبعيات:** S1-T-001, S1-T-002
**الناتج:** CRUD كامل للـ instances

---

### S1-T-005 | BullMQ — Instance Provisioning Queue
**المسؤول:** Backend Dev
**الوقت:** 6 ساعات | **النقاط:** 5

- [ ] `src/queues/index.ts` — Queue setup مع Redis
- [ ] `src/queues/instance.queue.ts`:
  - `addProvisionJob(instanceId, nodeId)`
  - `addDestroyJob(instanceId)`
- [ ] `src/workers/instance.worker.ts`:
  - Provision:
    1. SSH → check node ready
    2. SSH → setup openclaw config for this instance
    3. SSH → start gateway مع port فريد
    4. WebSocket → verify health
    5. Update DB: status → running
    6. Emit WebSocket event
  - Destroy:
    1. SSH → stop gateway
    2. SSH → cleanup config/data
    3. Update DB: status → destroyed
- [ ] Progress events → WebSocket → Layer 1
- [ ] Retry strategy: 3 retries مع exponential backoff

**التبعيات:** S1-T-001, S1-T-002
**الناتج:** provisioning يعمل async مع تحديثات real-time

---

### S1-T-006 | WebSocket Server (Socket.io)
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] `src/ws/index.ts` — Socket.io server مع JWT auth
- [ ] Rooms: `user:<id>`, `instance:<id>`, `admin`
- [ ] Auto-join: المستخدم ينضم لغرفه تلقائياً عند الاتصال
- [ ] Helper: `emitToUser(userId, event, data)`
- [ ] Helper: `emitToInstance(instanceId, event, data)`
- [ ] Helper: `emitToAdmins(event, data)`
- [ ] دمج مع BullMQ worker لإرسال progress events

**التبعيات:** S0-T-003 (JWT)
**الناتج:** داشبورد يستقبل أحداث real-time

---

### S1-T-007 | Instance Logs Endpoint
**المسؤول:** Backend Dev
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] `GET /instances/:id/logs?lines=100&since=2026-02-28`
- [ ] SSH → `tail -n 100 /tmp/openclaw-gateway.log`
- [ ] Streaming option مستقبلاً (SSE)

**التبعيات:** S1-T-001
**الناتج:** المستخدم يشوف سجلات instance

---

### S1-T-008 | Plans Seed + GET Endpoint
**المسؤول:** Backend Dev
**الوقت:** 2 ساعة | **النقاط:** 2

- [ ] Seed default plans (free, starter, pro)
- [ ] `GET /billing/plans` → قائمة الخطط المتاحة
- [ ] ربط plan بالـ user عند التسجيل (default = free)

**التبعيات:** S0-T-002
**الناتج:** خطط موجودة + كل مستخدم جديد على الخطة المجانية

---

### S1-T-009 | Integration Tests
**المسؤول:** أي مطور
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] اختبار instance lifecycle: create → start → status → stop → delete
- [ ] اختبار node service (مع mock SSH)
- [ ] اختبار gateway client (مع mock WebSocket)
- [ ] اختبار BullMQ workers
- [ ] اختبار WebSocket events

**التبعيات:** كل المهام أعلاه
**الناتج:** تغطية اختبارية للتدفقات الأساسية

---

### S1-T-010 | Deploy to Server
**المسؤول:** DevOps / Backend Lead
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] إعداد Nginx reverse proxy → port 4000
- [ ] PM2 لتشغيل الـ API
- [ ] `.env` على السيرفر
- [ ] Deploy script أو CI pipeline
- [ ] Smoke test: health + register + login + create instance

**التبعيات:** S0 + server access
**الناتج:** API يعمل على `http://45.55.253.17:4000`

---

## ما خارج Sprint 1

- ❌ ربط القنوات (Sprint 2)
- ❌ Billing/Stripe (Sprint 3)
- ❌ Admin panel كامل (Sprint 4)
- ❌ DigitalOcean API provisioning (مستقبل — الآن nodes مُعدّة يدوياً)

---

## نقاط Sprint 1

| المهمة | النقاط |
|--------|--------|
| S1-T-001 (SSH Controller) | 5 |
| S1-T-002 (Gateway Client) | 5 |
| S1-T-003 (Node CRUD) | 3 |
| S1-T-004 (Instance CRUD) | 5 |
| S1-T-005 (BullMQ) | 5 |
| S1-T-006 (WebSocket) | 3 |
| S1-T-007 (Logs) | 2 |
| S1-T-008 (Plans seed) | 2 |
| S1-T-009 (Tests) | 3 |
| S1-T-010 (Deploy) | 2 |
| **المجموع** | **35** |
