# Layer 2 — Database Schema (PostgreSQL + Drizzle)

## نظرة عامة

```
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Database                          │
│                                                                 │
│  ┌─────────┐  ┌───────────┐  ┌──────────┐  ┌───────────────┐  │
│  │  users   │  │ instances │  │ channels │  │ subscriptions │  │
│  └────┬────┘  └─────┬─────┘  └────┬─────┘  └───────┬───────┘  │
│       │             │              │                │           │
│  ┌────┴────┐  ┌─────┴─────┐  ┌────┴─────┐  ┌──────┴──────┐   │
│  │ sessions│  │   nodes   │  │  usage   │  │  invoices   │   │
│  └─────────┘  └───────────┘  └──────────┘  └─────────────┘   │
│                                                                 │
│  ┌───────────┐  ┌──────────────┐  ┌─────────────────────┐     │
│  │audit_logs │  │announcements │  │ system_settings     │     │
│  └───────────┘  └──────────────┘  └─────────────────────┘     │
└─────────────────────────────────────────────────────────────────┘
```

---

## الجداول

### 1. `users` — المستخدمين

```sql
CREATE TABLE users (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email           VARCHAR(255) UNIQUE NOT NULL,
  password_hash   VARCHAR(255) NOT NULL,
  name            VARCHAR(100),
  role            VARCHAR(20) DEFAULT 'user',    -- 'user' | 'admin' | 'superadmin'
  status          VARCHAR(20) DEFAULT 'pending', -- 'pending' | 'active' | 'suspended'
  email_verified  BOOLEAN DEFAULT FALSE,
  avatar_url      TEXT,

  -- Stripe
  stripe_customer_id  VARCHAR(100) UNIQUE,

  -- Metadata
  last_login_at   TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_stripe ON users(stripe_customer_id);
```

#### Drizzle Schema:
```typescript
export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).unique().notNull(),
  passwordHash: varchar('password_hash', { length: 255 }).notNull(),
  name: varchar('name', { length: 100 }),
  role: varchar('role', { length: 20 }).default('user'),
  status: varchar('status', { length: 20 }).default('pending'),
  emailVerified: boolean('email_verified').default(false),
  avatarUrl: text('avatar_url'),
  stripeCustomerId: varchar('stripe_customer_id', { length: 100 }).unique(),
  lastLoginAt: timestamp('last_login_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow(),
});
```

---

### 2. `refresh_tokens` — توكنات التجديد

```sql
CREATE TABLE refresh_tokens (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token_hash  VARCHAR(255) UNIQUE NOT NULL,
  expires_at  TIMESTAMPTZ NOT NULL,
  revoked     BOOLEAN DEFAULT FALSE,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_refresh_tokens_user ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_hash ON refresh_tokens(token_hash);
```

---

### 3. `email_verifications` — تأكيد البريد

```sql
CREATE TABLE email_verifications (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token       VARCHAR(255) UNIQUE NOT NULL,
  expires_at  TIMESTAMPTZ NOT NULL,
  used        BOOLEAN DEFAULT FALSE,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 4. `nodes` — عقد السيرفرات (VPS)

كل node هو سيرفر (Droplet) يشغّل openclaw.

```sql
CREATE TABLE nodes (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  label           VARCHAR(100),                    -- اسم ودّي
  ip_address      INET NOT NULL,
  ssh_port        INTEGER DEFAULT 22,
  ssh_key_id      VARCHAR(100),                    -- مفتاح SSH المرتبط

  -- DigitalOcean
  droplet_id      VARCHAR(50),                     -- DO Droplet ID
  region          VARCHAR(20) DEFAULT 'nyc1',
  size            VARCHAR(30) DEFAULT 's-1vcpu-1gb',

  -- OpenClaw Gateway
  gateway_port    INTEGER DEFAULT 18789,
  gateway_token   VARCHAR(255),                    -- auth token للاتصال
  openclaw_version VARCHAR(30),

  -- Status
  status          VARCHAR(20) DEFAULT 'provisioning',
  -- 'provisioning' | 'ready' | 'running' | 'stopped' | 'error' | 'destroying'

  last_health_at  TIMESTAMPTZ,
  error_message   TEXT,

  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_nodes_status ON nodes(status);
CREATE INDEX idx_nodes_droplet ON nodes(droplet_id);
```

---

### 5. `instances` — مثيلات openclaw (per user)

كل instance تربط مستخدم بـ node.

```sql
CREATE TABLE instances (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  node_id         UUID REFERENCES nodes(id) ON DELETE SET NULL,

  name            VARCHAR(100) NOT NULL,           -- "مساعد متجري"
  slug            VARCHAR(100) UNIQUE NOT NULL,    -- URL-safe identifier

  -- Config
  agent_name      VARCHAR(100),                    -- اسم الـ agent المربوط
  config_snapshot JSONB,                           -- آخر config مطبّق

  -- Status
  status          VARCHAR(20) DEFAULT 'creating',
  -- 'creating' | 'running' | 'stopped' | 'error' | 'suspended'

  -- Plan limits
  plan_id         UUID REFERENCES plans(id),

  started_at      TIMESTAMPTZ,
  stopped_at      TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_instances_user ON instances(user_id);
CREATE INDEX idx_instances_node ON instances(node_id);
CREATE INDEX idx_instances_status ON instances(status);
```

---

### 6. `channels` — القنوات المربوطة

```sql
CREATE TABLE channels (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  instance_id     UUID NOT NULL REFERENCES instances(id) ON DELETE CASCADE,

  type            VARCHAR(30) NOT NULL,  -- 'whatsapp' | 'telegram' | 'discord' | ...
  account_id      VARCHAR(255),          -- channel-specific account (phone, bot token hash)
  display_name    VARCHAR(100),          -- "@MyBot" أو رقم الهاتف المخفي

  status          VARCHAR(20) DEFAULT 'disconnected',
  -- 'connecting' | 'connected' | 'disconnected' | 'error' | 'logged_out'

  config          JSONB,                 -- channel-specific settings
  last_active_at  TIMESTAMPTZ,
  error_message   TEXT,

  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),

  UNIQUE(instance_id, type, account_id)
);

CREATE INDEX idx_channels_instance ON channels(instance_id);
CREATE INDEX idx_channels_type ON channels(type);
```

---

### 7. `plans` — خطط الاشتراك

```sql
CREATE TABLE plans (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name            VARCHAR(50) NOT NULL,            -- 'free' | 'starter' | 'pro' | 'enterprise'
  display_name    VARCHAR(100) NOT NULL,           -- "خطة المبتدئ"

  -- Limits
  max_instances       INTEGER DEFAULT 1,
  max_channels        INTEGER DEFAULT 2,
  max_messages_month  INTEGER DEFAULT 1000,

  -- Pricing
  price_monthly   DECIMAL(10,2) DEFAULT 0,
  price_yearly    DECIMAL(10,2) DEFAULT 0,
  currency        VARCHAR(3) DEFAULT 'USD',
  stripe_price_id_monthly VARCHAR(100),
  stripe_price_id_yearly  VARCHAR(100),

  -- Flags
  is_active       BOOLEAN DEFAULT TRUE,
  is_default      BOOLEAN DEFAULT FALSE,           -- خطة المسجّل الجديد

  features        JSONB,                           -- { "priority_support": true, ... }

  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 8. `subscriptions` — اشتراكات المستخدمين

```sql
CREATE TABLE subscriptions (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  plan_id             UUID NOT NULL REFERENCES plans(id),

  stripe_subscription_id VARCHAR(100) UNIQUE,

  status              VARCHAR(20) DEFAULT 'active',
  -- 'active' | 'past_due' | 'canceled' | 'trialing'

  billing_cycle       VARCHAR(10) DEFAULT 'monthly', -- 'monthly' | 'yearly'
  current_period_start TIMESTAMPTZ,
  current_period_end   TIMESTAMPTZ,
  cancel_at            TIMESTAMPTZ,

  created_at          TIMESTAMPTZ DEFAULT NOW(),
  updated_at          TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_subscriptions_user ON subscriptions(user_id);
CREATE INDEX idx_subscriptions_stripe ON subscriptions(stripe_subscription_id);
```

---

### 9. `invoices` — الفواتير

```sql
CREATE TABLE invoices (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id             UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  subscription_id     UUID REFERENCES subscriptions(id),

  stripe_invoice_id   VARCHAR(100) UNIQUE,

  amount              DECIMAL(10,2) NOT NULL,
  currency            VARCHAR(3) DEFAULT 'USD',
  status              VARCHAR(20) DEFAULT 'pending',
  -- 'pending' | 'paid' | 'failed' | 'refunded'

  pdf_url             TEXT,
  paid_at             TIMESTAMPTZ,

  created_at          TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_invoices_user ON invoices(user_id);
```

---

### 10. `usage_records` — سجل الاستخدام

```sql
CREATE TABLE usage_records (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  instance_id     UUID NOT NULL REFERENCES instances(id) ON DELETE CASCADE,
  user_id         UUID NOT NULL REFERENCES users(id),

  -- Period
  period_start    DATE NOT NULL,
  period_end      DATE NOT NULL,

  -- Metrics
  messages_sent       INTEGER DEFAULT 0,
  messages_received   INTEGER DEFAULT 0,
  api_calls           INTEGER DEFAULT 0,
  tokens_used         BIGINT DEFAULT 0,

  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),

  UNIQUE(instance_id, period_start)
);

CREATE INDEX idx_usage_instance_period ON usage_records(instance_id, period_start);
CREATE INDEX idx_usage_user ON usage_records(user_id);
```

---

### 11. `audit_logs` — سجل المراجعة

```sql
CREATE TABLE audit_logs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  actor_id        UUID REFERENCES users(id),       -- من قام بالعملية
  actor_role      VARCHAR(20),

  action          VARCHAR(50) NOT NULL,             -- 'instance.create' | 'user.suspend' | ...
  resource_type   VARCHAR(30),                      -- 'instance' | 'user' | 'channel' | ...
  resource_id     UUID,

  details         JSONB,                            -- بيانات إضافية
  ip_address      INET,

  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_audit_actor ON audit_logs(actor_id);
CREATE INDEX idx_audit_action ON audit_logs(action);
CREATE INDEX idx_audit_resource ON audit_logs(resource_type, resource_id);
CREATE INDEX idx_audit_created ON audit_logs(created_at);
```

---

### 12. `announcements` — الإعلانات

```sql
CREATE TABLE announcements (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  title           VARCHAR(200) NOT NULL,
  body            TEXT NOT NULL,
  type            VARCHAR(20) DEFAULT 'info',       -- 'info' | 'warning' | 'maintenance'

  is_active       BOOLEAN DEFAULT TRUE,
  starts_at       TIMESTAMPTZ DEFAULT NOW(),
  ends_at         TIMESTAMPTZ,

  created_by      UUID REFERENCES users(id),
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

---

### 13. `system_settings` — إعدادات النظام

```sql
CREATE TABLE system_settings (
  key             VARCHAR(100) PRIMARY KEY,
  value           JSONB NOT NULL,
  description     TEXT,
  updated_by      UUID REFERENCES users(id),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);
```

---

## العلاقات (ER Summary)

```
users ──1:N──► instances ──1:N──► channels
  │                │
  │                └──N:1──► nodes
  │
  ├──1:N──► subscriptions ──N:1──► plans
  │              │
  │              └──1:N──► invoices
  │
  ├──1:N──► refresh_tokens
  ├──1:N──► email_verifications
  ├──1:N──► usage_records
  └──1:N──► audit_logs
```

---

## ملاحظات

- **JSONB** يُستخدم لـ config مرنة (channel settings, plan features) — لا نحتاج جدول لكل إعداد.
- **UUID** لكل المفاتيح — آمن وغير قابل للتخمين.
- **Soft delete غير مطلوب** في البداية — نحذف فعلياً مع cascade.
- **Timestamps** بـ timezone لدعم مستخدمين من مناطق مختلفة.
- **Indexes** على كل الأعمدة المستخدمة في WHERE و JOIN.
