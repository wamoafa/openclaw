# Sprint 2 — Channels + Gateway Control

**المدة:** أسبوعان (10 أيام)
**الهدف:** المستخدم يربط WhatsApp/Telegram/Discord من الداشبورد ويتحكم بإعدادات openclaw

---

## متى ينتهي Sprint 2 بنجاح؟

- [ ] ربط WhatsApp: QR code يظهر في الداشبورد → المستخدم يمسح → القناة تتصل
- [ ] ربط Telegram: إدخال Bot Token → القناة تشتغل
- [ ] ربط Discord: إدخال Bot Token → القناة تشتغل
- [ ] `GET /instances/:id/channels` → قائمة القنوات مع حالتها المباشرة
- [ ] فصل قناة يعمل
- [ ] تعديل إعدادات الـ agent (اسم، system prompt) من الداشبورد

---

## المهام

### S2-T-001 | Channel Service — ربط القنوات عبر Gateway RPC
**المسؤول:** Backend Dev
**الوقت:** 8 ساعات | **النقاط:** 5

- [ ] `src/services/channel.service.ts`:
  - `linkWhatsApp(instanceId)` → config.set whatsapp + restart + monitor QR
  - `linkTelegram(instanceId, botToken)` → config.set telegram.token + restart
  - `linkDiscord(instanceId, botToken, guildId)` → config.set discord + restart
  - `unlinkChannel(instanceId, type)` → config.set enabled=false + restart
  - `getChannelStatus(instanceId)` → Gateway RPC: channels.status
  - `reconnectChannel(instanceId, channelId)` → restart channel
- [ ] دعم كل القنوات الأساسية: whatsapp, telegram, discord, slack, signal
- [ ] Gateway RPC calls:
  - `config.set` لتعديل channel config
  - `config.apply` لتطبيق config كامل
  - restart عبر SSH بعد التعديل

**التبعيات:** S1-T-002 (Gateway Client)
**الناتج:** كل القنوات قابلة للربط/الفصل برمجياً

---

### S2-T-002 | WhatsApp QR Flow (WebSocket)
**المسؤول:** Backend Dev
**الوقت:** 6 ساعات | **النقاط:** 5

- [ ] تدفق ربط WhatsApp:
  1. User → `POST /instances/:id/channels` (type: whatsapp)
  2. Layer 2 → SSH: enable whatsapp + restart gateway
  3. Layer 2 → Gateway RPC: subscribe to events
  4. Gateway يُولّد QR code
  5. Layer 2 → WebSocket → Layer 1: `channel:qr` event مع base64 QR
  6. User يمسح QR من WhatsApp
  7. Gateway يُبلّغ بالاتصال
  8. Layer 2 → DB: channel status = connected
  9. Layer 2 → WebSocket: `channel:linked` event
- [ ] `GET /instances/:id/channels/whatsapp/qr` — polling fallback
- [ ] QR refresh: إذا انتهت صلاحية QR، أرسل واحد جديد
- [ ] Timeout: إذا لم يمسح خلال 5 دقائق → cleanup

**التبعيات:** S2-T-001, S1-T-006 (WebSocket)
**الناتج:** تدفق WhatsApp QR يعمل end-to-end

---

### S2-T-003 | Channel Routes
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] `src/routes/channels.ts`:
  - `GET /instances/:id/channels` → قائمة مع حالة مباشرة
  - `POST /instances/:id/channels` → ربط قناة جديدة
  - `GET /instances/:id/channels/:channelId` → تفاصيل
  - `DELETE /instances/:id/channels/:channelId` → فصل
  - `POST /instances/:id/channels/:channelId/reconnect` → إعادة اتصال
  - `GET /instances/:id/channels/whatsapp/qr` → QR code
- [ ] Validation: Zod schemas لكل request
- [ ] Authorization: owner check لكل endpoint

**التبعيات:** S2-T-001
**الناتج:** Channel API كامل

---

### S2-T-004 | Agent Config Management
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] `src/services/config.service.ts`:
  - `getConfig(instanceId)` → Gateway RPC: config.get
  - `updateConfig(instanceId, patch)` → Gateway RPC: config.set / config.apply
  - `getAgentConfig(instanceId)` → agent name, system prompt, model
  - `updateAgentConfig(instanceId, data)` → تعديل agent settings
- [ ] Routes:
  - `GET /instances/:id/config` → الإعدادات الحالية
  - `PATCH /instances/:id/config` → تعديل إعدادات
  - `GET /instances/:id/config/agent` → إعدادات الـ agent
  - `PATCH /instances/:id/config/agent` → تعديل الـ agent
- [ ] Config snapshot: حفظ آخر config في DB

**التبعيات:** S1-T-002 (Gateway Client)
**الناتج:** المستخدم يعدّل إعدادات openclaw من الداشبورد

---

### S2-T-005 | Channel Status Monitoring
**المسؤول:** Backend Dev
**الوقت:** 4 ساعات | **النقاط:** 3

- [ ] BullMQ recurring job: فحص حالة القنوات كل 30 ثانية
- [ ] `src/workers/monitoring.worker.ts`:
  - لكل instance نشط: Gateway RPC → channels.status
  - مقارنة مع الحالة المخزّنة في DB
  - إذا تغيّرت → تحديث DB + WebSocket event
- [ ] اكتشاف القنوات المقطوعة (disconnected/logged_out)
- [ ] إرسال تنبيه عبر WebSocket إذا قناة سقطت

**التبعيات:** S1-T-002, S1-T-005 (BullMQ)
**الناتج:** حالة القنوات محدّثة دائماً

---

### S2-T-006 | Session Management API
**المسؤول:** Backend Dev
**الوقت:** 3 ساعات | **النقاط:** 3

- [ ] Routes:
  - `GET /instances/:id/sessions` → Gateway RPC: sessions.list
  - `GET /instances/:id/sessions/:sessionId` → تفاصيل جلسة
  - `POST /instances/:id/sessions/:sessionId/reset` → إعادة تعيين
  - `DELETE /instances/:id/sessions/:sessionId` → حذف جلسة
- [ ] عرض عدد الجلسات النشطة في instance status

**التبعيات:** S1-T-002
**الناتج:** المستخدم يدير جلسات المحادثة

---

### S2-T-007 | Channel Linking Queue (BullMQ)
**المسؤول:** Backend Dev
**الوقت:** 3 ساعات | **النقاط:** 3

- [ ] `src/queues/channel.queue.ts`:
  - `addLinkJob(instanceId, channelType, config)`
  - `addUnlinkJob(instanceId, channelType)`
- [ ] `src/workers/channel.worker.ts`:
  - Link: config.set → restart → verify → update DB → emit event
  - Unlink: config.set → restart → update DB → emit event
- [ ] Error handling: إذا فشل الربط → cleanup + error event

**التبعيات:** S2-T-001, S1-T-005
**الناتج:** ربط القنوات يعمل async مع retry

---

### S2-T-008 | Integration Tests — Channels
**المسؤول:** أي مطور
**الوقت:** 3 ساعات | **النقاط:** 2

- [ ] اختبار channel service (مع mock Gateway RPC)
- [ ] اختبار WhatsApp QR flow
- [ ] اختبار channel status monitoring
- [ ] اختبار config updates

**التبعيات:** كل المهام أعلاه
**الناتج:** تغطية اختبارية لتدفقات القنوات

---

## نقاط Sprint 2

| المهمة | النقاط |
|--------|--------|
| S2-T-001 (Channel Service) | 5 |
| S2-T-002 (WhatsApp QR) | 5 |
| S2-T-003 (Channel Routes) | 3 |
| S2-T-004 (Agent Config) | 3 |
| S2-T-005 (Status Monitoring) | 3 |
| S2-T-006 (Sessions API) | 3 |
| S2-T-007 (Channel Queue) | 3 |
| S2-T-008 (Tests) | 2 |
| **المجموع** | **27** |
