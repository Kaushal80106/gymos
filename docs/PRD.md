# GymOS: Product Requirements Document (PRD)

| | |
|---|---|
| **Product** | GymOS: gym management, collections and retention platform |
| **Version** | 1.0 (MVP scope) |
| **Date** | 6 October 2026 |
| **Status** | Draft for review |
| **Stack** | Next.js (frontend), NestJS (backend), PostgreSQL, Redis |
| **Market** | India (first), independent gyms and fitness studios |

---

## 1. Overview

### 1.1 Summary
GymOS is a multi-tenant SaaS for independent gyms. It manages members, memberships, attendance and billing, and automates **renewal reminders and payment collection over WhatsApp**. A dashboard gives owners a live picture of revenue, pending dues, expiring members and at-risk members. A chatbot answers common member questions and hands off to staff when needed.

### 1.2 Problem statement
Independent gyms track members in registers, Excel and WhatsApp. As a result:
- Renewals are missed because nobody knows who expires this week.
- Pending fees go uncollected because follow-up is manual and awkward.
- Owners cannot see revenue, churn or attendance trends.
- Front-desk staff spend hours answering repetitive questions (timings, fees, balance).
- Inactive members quietly churn before anyone notices.

### 1.3 Vision
"Never lose a membership renewal again." GymOS should pay for itself every month through recovered renewals and dues.

### 1.4 Goals
| # | Goal | Measure |
|---|---|---|
| G1 | Increase renewal rate | +10 percentage points within 3 months of use |
| G2 | Reduce pending dues | 30% reduction in dues older than 7 days |
| G3 | Save front-desk time | Check-in under 5 seconds; renewal under 30 seconds |
| G4 | Fast onboarding | New gym live in under 30 minutes (with CSV import) |
| G5 | Prove ROI | Dashboard shows "₹ recovered by GymOS" per month |

### 1.5 Non-goals (v1)
- Payroll, full accounting or a general ledger (provide CSV/Tally-friendly exports instead)
- Hardware sales; biometric devices are integrated, not sold
- Workout-app features such as exercise video libraries
- Marketplace, multi-gym member passes
- Native mobile apps (the PWA covers this)
- A general-purpose AI assistant (the chatbot is scoped to gym tasks)

---

## 2. Users and personas

| Persona | Role | Key needs |
|---|---|---|
| **Owner (Ravi, 38)** | Runs 1-2 gyms | Collection visibility, renewal numbers, daily digest, controls spending |
| **Manager** | Operations | Dues follow-up, staff tasks, reports |
| **Front desk (Priya, 24)** | Check-ins, joins, payments | Speed, simple screens, few clicks, works on phone or tablet |
| **Trainer** | Coaching | Assigned members, attendance alerts, notes |
| **Member (Arjun, 28)** | End user | WhatsApp reminders, easy UPI payment, receipts, freeze requests |

---

## 3. Scope and release plan

### 3.1 MVP (Release 1, weeks 1-12)
Members, plans, memberships, attendance, billing and payments, dues tracker, reminder engine (WhatsApp), dashboard, roles, daily owner digest, CSV import/export.

### 3.2 Release 2
Chatbot (member and owner), unified inbox, lead pipeline, trainer module, PT packs, member PWA, expense tracking, multi-branch.

### 3.3 Release 3
Class scheduling and booking, UPI AutoPay, locker management, supplements/inventory, body measurements, referral and loyalty, regional-language bot, churn prediction, biometric integration.

---

## 4. Functional requirements

Priority: **P0** = MVP must-have, **P1** = Release 2, **P2** = later.

### 4.1 Authentication, tenants and roles (P0)
| ID | Requirement |
|---|---|
| AUTH-1 | Owner signs up with phone number + OTP (email optional). Creates a gym (tenant) and a default branch. |
| AUTH-2 | Roles: Owner, Manager, Front Desk, Trainer. Role-based access control (RBAC) enforced in the API on every endpoint. |
| AUTH-3 | Owner can invite staff by phone/email, assign a role and a branch, and deactivate them. |
| AUTH-4 | Sessions use short-lived JWT access tokens with rotating refresh tokens; staff can log out of all devices. |
| AUTH-5 | Every record is scoped to a tenant; a user can never read or write another tenant's data. |
| AUTH-6 | Audit log of sensitive actions (payment edits, waivers, role changes, data exports). |

**Permissions matrix (summary)**

| Capability | Owner | Manager | Front Desk | Trainer |
|---|:-:|:-:|:-:|:-:|
| Dashboard (full finance) | ✔ | ✔ | limited | ✖ |
| Add/edit members | ✔ | ✔ | ✔ | ✖ |
| Check-in | ✔ | ✔ | ✔ | ✔ |
| Record payment | ✔ | ✔ | ✔ | ✖ |
| Waive/reverse payment | ✔ | ✔ (limit) | ✖ | ✖ |
| Plans and pricing | ✔ | ✔ | ✖ | ✖ |
| Reminder rules/templates | ✔ | ✔ | ✖ | ✖ |
| Staff and billing settings | ✔ | ✖ | ✖ | ✖ |
| Export data | ✔ | ✔ | ✖ | ✖ |

### 4.2 Member management (P0)
| ID | Requirement |
|---|---|
| MEM-1 | Create a member with name, phone (unique per tenant), DOB, gender, photo, emergency contact, goal, health notes and join source. |
| MEM-2 | Search by name or phone, instant results (type-ahead) with status filters. |
| MEM-3 | Member profile shows plan, expiry, attendance streak, payments, dues, messages, notes and assigned trainer. |
| MEM-4 | Member statuses: Lead, Trial, Active, Expiring, Expired, Frozen, Lapsed. Derived automatically from memberships. |
| MEM-5 | WhatsApp consent flag captured at creation; opt-out respected system-wide. |
| MEM-6 | Bulk import from CSV with validation, duplicate detection and an error report. |
| MEM-7 | Tags and custom fields (limited set in MVP). |
| MEM-8 | Soft delete and anonymise on request (privacy compliance). |

### 4.3 Plans and memberships (P0)
| ID | Requirement |
|---|---|
| PLN-1 | Owner defines plans: name, duration (days/months), price, type (membership / PT pack / day pass), joining fee, tax rate. |
| PLN-2 | Assign a plan to a member with a start date; end date is computed. |
| PLN-3 | **Renewal rule:** renewing before expiry extends from the existing end date; after expiry (past grace) it starts from the chosen date. |
| PLN-4 | **Freeze:** member can freeze for N days; end date extends by the frozen days. Per-plan limit on number and length of freezes. Requires staff approval. |
| PLN-5 | **Upgrade/transfer:** pro-rated credit calculation shown before confirmation. |
| PLN-6 | Configurable **grace period** (days after expiry before check-in is blocked). |
| PLN-7 | Nightly job updates statuses (Active → Expiring → Expired → Lapsed) using the branch's timezone. |

### 4.4 Attendance and check-in (P0)
| ID | Requirement |
|---|---|
| ATT-1 | Front-desk check-in by phone/name search, member QR scan or member self-QR at the entrance. |
| ATT-2 | Check-in screen shows photo, plan status banner (green/amber/red), days left and dues. |
| ATT-3 | Expired or blocked members show a clear warning with a one-tap "Renew" action; staff may override with a reason (logged). |
| ATT-4 | Duplicate check-in within a configurable window (e.g. 30 min) is ignored. |
| ATT-5 | Attendance history, streaks and "days since last visit" stored per member. |
| ATT-6 | (P2) Biometric/face device integration via webhook or SDK bridge. |

### 4.5 Billing and payments (P0)
| ID | Requirement |
|---|---|
| BIL-1 | Create an invoice automatically on joining or renewal; supports discount, joining fee and GST (CGST/SGST/IGST). |
| BIL-2 | Record payments by cash, UPI, card, bank transfer or online link. Supports **partial payments and instalments**. |
| BIL-3 | Payment records are **append-only**. Corrections are made through reversals/credit notes, never edits. |
| BIL-4 | Generate a PDF receipt/invoice with the gym's branding; send via WhatsApp/email on payment. |
| BIL-5 | **Online payment link** (Razorpay/Cashfree, UPI-first). Gateway webhook marks the invoice paid, idempotently. |
| BIL-6 | Invoice numbering is sequential per tenant and financial year, with no gaps. |
| BIL-7 | Daily cash-closing report per branch and staff member. |
| BIL-8 | Export to CSV/Excel for accountants (and a Tally-compatible format in Release 2). |

### 4.6 Dues tracker (P0)
| ID | Requirement |
|---|---|
| DUE-1 | Dues board lists all unpaid/partially-paid invoices with ageing buckets: not due, 0-7, 8-30, 30+ days. |
| DUE-2 | Per-row actions: collect payment, send reminder, call (tel: link), add note, set promise-to-pay date. |
| DUE-3 | Bulk "send reminder" with a message preview and a confirmation step. |
| DUE-4 | Owner can write off or waive a due with a mandatory reason (audited). |
| DUE-5 | Total outstanding and recovered-this-month are shown on the dashboard. |

### 4.7 Reminder and automation engine (P0), core differentiator
| ID | Requirement |
|---|---|
| REM-1 | Rule-based triggers: X days before expiry, on expiry, X days after expiry, instalment due, overdue by X days, payment received, birthday, inactive for X days. |
| REM-2 | Default rule set provided at onboarding; owner can edit timing, enable or disable, and choose channel (WhatsApp, SMS, email). |
| REM-3 | Templates support variables (`{{name}}`, `{{expiry_date}}`, `{{amount_due}}`, `{{pay_link}}`, `{{gym_name}}`). WhatsApp templates are submitted for Meta approval via the BSP. |
| REM-4 | Guardrails: quiet hours (default 9 pm to 9 am), maximum messages per member per week, stop on payment/renewal, and respect opt-out. |
| REM-5 | A reminder is sent at most once per (member, rule, cycle). Idempotency keys prevent duplicates after retries. |
| REM-6 | Delivery tracking: queued, sent, delivered, read, failed (via webhooks) with retry and backoff. |
| REM-7 | Owner sees "₹ recovered via reminders" (payments received within 7 days of a reminder, attributed). |
| REM-8 | **Daily owner digest** at a configurable time (e.g. 8 am): expiring, dues, yesterday's collection, at-risk members. |

### 4.8 Dashboard (P0)
| Section | Widgets |
|---|---|
| Today | Check-ins, new joins, collection, renewals due today |
| Retention | Expiring in 7/15/30 days, renewal rate, churn rate, at-risk (no visit 7+ days) |
| Money | Revenue (day/month/custom), pending dues by ageing, collection by mode, plan-wise revenue, recovered by reminders |
| Growth | Leads by source, lead-to-join conversion (R2) |
| Operations | Peak-hours heatmap, trainer-wise active members (R2) |

Requirements: date-range filter, branch filter, load in under 2 seconds (cached aggregates in Redis), and each tile clickable to the underlying list.

### 4.9 Communication hub (P1)
| ID | Requirement |
|---|---|
| COM-1 | Unified inbox: WhatsApp threads per member with message status, assignment to staff and internal notes. |
| COM-2 | 24-hour customer-service window awareness: free-form replies inside the window, template messages outside it. |
| COM-3 | Bulk broadcast to segments (expiring, expired, birthday, trial, custom tags) using approved templates. |
| COM-4 | Email and SMS as secondary channels. |
| COM-5 | Opt-in/opt-out handling ("STOP" keyword) recorded with timestamps. |

### 4.10 Chatbot (P1)
| ID | Requirement |
|---|---|
| BOT-1 | Intent layer (rules + classifier) for: timings, fees/plans, membership status, pending balance, send payment link, send receipt, join enquiry, freeze request. |
| BOT-2 | LLM + retrieval over the gym's own knowledge base (plans, timings, policies, FAQs) for open questions, with citations to the source field and a "cannot answer" fallback. |
| BOT-3 | **Safe actions only:** the bot can read data, send links and create requests/leads. It cannot change money records, waive dues or approve freezes. |
| BOT-4 | Human handoff on low confidence, repeated failure, negative sentiment or the keyword "agent". The thread is assigned to staff with context. |
| BOT-5 | Owner assistant: natural-language queries such as "who hasn't paid this month?" return a list; bulk actions require an explicit confirmation step. |
| BOT-6 | English, Hindi and Hinglish in R2; Marathi/Tamil and others later. |
| BOT-7 | All bot conversations are logged and reviewable; the owner can switch the bot off per gym or per thread. |

### 4.11 Leads (P1)
Lead capture from walk-in form, WhatsApp, Instagram/Facebook lead ads and a website embed form. Kanban stages: New, Contacted, Trial Booked, Trial Done, Joined, Lost. Follow-up tasks with reminders; conversion analytics by source.

### 4.12 Staff, tasks and settings
- Tasks and follow-ups with assignee, due date and member link (P1).
- Settings: gym profile and branding (logo, accent colour), branches, plans, tax (GST number), reminder rules, templates, working hours, grace period, freeze policy, roles.
- Subscription and billing page for the gym's own GymOS plan (P0 minimal).

### 4.13 Member PWA (P1)
Login via phone OTP; shows plan and expiry, dues, pay now, receipts, attendance, freeze request, class booking (R3) and the gym's QR for check-in.

---

## 5. Key user flows

1. **Onboarding:** sign up with OTP, create gym, add branch, choose plans, import members by CSV, enable default reminders, connect WhatsApp (guided), invite staff.
2. **New join:** search finds no match, add member, choose plan, invoice generated, take payment (full or partial), receipt sent on WhatsApp, QR issued.
3. **Daily check-in:** search or scan, status banner, enter. If expired, tap "Renew", pay, continue.
4. **Renewal automation:** the rule fires 7 days before expiry, sends a message with a UPI link, the member pays, the webhook marks the invoice paid, the membership extends, a receipt is sent, and the reminder sequence stops.
5. **Dues follow-up:** the owner opens the dues board, selects 12 overdue members, previews the message, confirms, and the queue sends within rate limits.
6. **Freeze:** the member asks the bot, a request is created, staff approve, the end date shifts, and the member is notified.

---

## 6. Technical architecture

### 6.1 Stack
| Layer | Choice | Notes |
|---|---|---|
| Frontend | **Next.js (App Router) + TypeScript** | Tailwind CSS + shadcn/ui, TanStack Query, React Hook Form + Zod, Recharts/ECharts, PWA support (next-pwa/Serwist) |
| Backend | **NestJS + TypeScript** (modular monolith) | REST + OpenAPI (Swagger); class-validator; Passport/JWT; BullMQ workers |
| Database | **PostgreSQL 16** | Prisma or TypeORM (Prisma recommended); row-level security; `pg_trgm` for member search |
| Cache / queue | **Redis 7** | BullMQ queues, rate limiting, session/refresh-token store, dashboard cache, idempotency keys |
| Object storage | S3-compatible (AWS S3 / Cloudflare R2) | Photos, invoice PDFs, imports/exports |
| Search | Postgres full-text + trigram (no extra service in MVP) | |
| Messaging | WhatsApp Cloud API / BSP (Gupshup, Twilio, 360dialog); SES/SendGrid for email; MSG91 for SMS/OTP | |
| Payments | Razorpay or Cashfree | Payment links, webhooks, later UPI AutoPay |
| AI (R2) | LLM API + pgvector for retrieval | Per-tenant knowledge base |
| Infra | Docker; AWS (ECS/Fargate or EC2) or Railway/Render to start; RDS Postgres; ElastiCache | CI/CD with GitHub Actions |
| Observability | Sentry, OpenTelemetry, Prometheus/Grafana or Datadog; structured JSON logs | |

### 6.2 High-level diagram
```
 Browser / PWA (Next.js)
          │ HTTPS
      CDN / WAF
          │
   NestJS API (REST)  ──── Redis (cache, rate limit, sessions)
   ├ Auth & Tenants
   ├ Members / Plans / Memberships
   ├ Attendance
   ├ Billing & Payments
   ├ Dues
   ├ Reminders & Templates
   ├ Communication / Chatbot (R2)
   ├ Reports / Dashboard
   └ Webhooks (WhatsApp, Payments)
          │
   PostgreSQL (RLS by tenant_id)        BullMQ Workers (separate process)
          │                              ├ reminder-scheduler (cron + events)
   S3 (files)                            ├ message-sender
                                         ├ webhook-processor
                                         ├ pdf/report generator
                                         └ nightly status updater
```

### 6.3 Architecture decisions
| Decision | Rationale |
|---|---|
| **Modular monolith** | One deployable, clear module boundaries (Nest modules), splittable later. |
| **Shared DB, shared schema, `tenant_id` + RLS** | Cheapest at thousands of small tenants. The tenant is set per request via `SET LOCAL app.tenant_id`, and Postgres policies enforce isolation. |
| **Separate worker process** | Reminders and webhooks must not slow the API; scale workers independently. |
| **Event-driven automation** | Domain events (`invoice.created`, `payment.received`, `membership.expiring`) are handled by listeners that enqueue jobs. |
| **Idempotency everywhere money moves** | `Idempotency-Key` header and unique constraints on gateway references. |
| **Append-only payments + audit log** | Trust and traceability. |
| **Timezone-aware** | Store UTC; compute expiry and reminder windows in the branch timezone (default Asia/Kolkata). |
| **Config over code** | Reminder rules, plans and templates are data, so new verticals need no forks. |

### 6.4 Repository layout (monorepo)
```
gymos/
  apps/
    web/          # Next.js
    api/          # NestJS API
    worker/       # NestJS standalone context (BullMQ processors)
  packages/
    shared/       # Zod schemas, types, constants
    ui/           # shared components / design tokens
  infra/          # Docker, Terraform / compose, CI
```
Use pnpm + Turborepo (or Nx).

### 6.5 Core data model (PostgreSQL)

All tenant tables include `tenant_id UUID NOT NULL`, `created_at`, `updated_at` and `deleted_at` (where soft delete applies).

| Table | Key columns |
|---|---|
| `tenants` | id, name, slug, gstin, plan, status, branding (jsonb), timezone |
| `branches` | id, tenant_id, name, address, phone, timezone |
| `users` | id, tenant_id, name, phone, email, role, branch_ids, status |
| `members` | id, tenant_id, branch_id, name, phone, dob, gender, photo_url, goal, source, status, trainer_id, consent_whatsapp, consent_at, custom (jsonb) |
| `plans` | id, tenant_id, name, type, duration_days, price, joining_fee, tax_rate, freeze_policy (jsonb), active |
| `memberships` | id, tenant_id, member_id, plan_id, start_date, end_date, status, frozen_days, balance_due |
| `freezes` | id, membership_id, from_date, to_date, status, approved_by |
| `invoices` | id, tenant_id, number, member_id, membership_id, subtotal, discount, tax, total, status, due_date |
| `payments` | id, tenant_id, invoice_id, amount, mode, gateway_ref (unique), received_at, received_by, **append-only** |
| `payment_reversals` | id, payment_id, amount, reason, created_by |
| `attendance` | id, tenant_id, member_id, branch_id, checkin_at, method, override_reason |
| `reminder_rules` | id, tenant_id, trigger, offset_days, channel, template_id, active, conditions (jsonb) |
| `templates` | id, tenant_id, channel, name, body, variables, wa_template_id, approval_status |
| `reminder_log` | id, tenant_id, member_id, rule_id, cycle_key (unique with member+rule), status, scheduled_at, sent_at |
| `messages` | id, tenant_id, member_id, thread_id, channel, direction, body, status, provider_id, sent_at |
| `leads` | id, tenant_id, name, phone, source, stage, assigned_to, next_followup_at |
| `tasks` | id, tenant_id, assignee_id, member_id, title, due_at, status |
| `audit_log` | id, tenant_id, user_id, action, entity, entity_id, before, after, ip, at |
| `knowledge_docs` | id, tenant_id, content, embedding (pgvector) |  *(R2)* |

**Indexes (examples):** `members(tenant_id, phone) UNIQUE`; `memberships(tenant_id, end_date, status)`; `invoices(tenant_id, status, due_date)`; `attendance(tenant_id, member_id, checkin_at DESC)`; `reminder_log(tenant_id, member_id, rule_id, cycle_key) UNIQUE`; trigram index on `members.name`.

### 6.6 Redis usage
| Use | Detail |
|---|---|
| BullMQ queues | `reminders`, `messages`, `webhooks`, `reports`, `status-jobs`, with retries, backoff and dead-letter handling |
| Rate limiting | Per IP and per tenant on the API; per-number WhatsApp send throttling |
| Dashboard cache | Aggregates keyed by `tenant:branch:range`, invalidated on events, TTL 60-300 s |
| Idempotency | Short-lived keys for webhook and payment requests |
| Sessions | Refresh-token allow-list and OTP attempt counters |

### 6.7 API design (REST, versioned `/api/v1`)

| Area | Endpoints (examples) |
|---|---|
| Auth | `POST /auth/otp/request`, `POST /auth/otp/verify`, `POST /auth/refresh`, `POST /auth/logout` |
| Members | `GET /members?search=&status=`, `POST /members`, `GET/PATCH /members/:id`, `POST /members/import` |
| Plans | `GET/POST /plans`, `PATCH /plans/:id` |
| Memberships | `POST /members/:id/memberships`, `POST /memberships/:id/renew`, `POST /memberships/:id/freeze`, `POST /memberships/:id/upgrade` |
| Attendance | `POST /attendance/check-in`, `GET /attendance?from=&to=` |
| Billing | `GET/POST /invoices`, `POST /invoices/:id/payments`, `POST /payments/:id/reverse`, `POST /invoices/:id/payment-link`, `GET /invoices/:id/pdf` |
| Dues | `GET /dues?bucket=`, `POST /dues/remind`, `POST /dues/:id/waive` |
| Reminders | `GET/POST /reminder-rules`, `GET/POST /templates`, `GET /reminder-log` |
| Dashboard | `GET /dashboard/summary`, `/dashboard/revenue`, `/dashboard/retention` |
| Webhooks | `POST /webhooks/whatsapp`, `POST /webhooks/payments` (signature-verified) |
| Messaging (R2) | `GET /threads`, `POST /threads/:id/messages`, `POST /broadcasts` |

Conventions: cursor pagination, consistent error format (`code`, `message`, `details`), `Idempotency-Key` on money endpoints, OpenAPI spec generated by Nest and consumed by a typed client in the frontend.

### 6.8 Reminder engine design
1. A **scheduler job** runs every 15 minutes per tenant timezone. It finds memberships and invoices matching each active rule's trigger window.
2. For each match it inserts a `reminder_log` row with a unique `(member, rule, cycle_key)`. The unique constraint makes it safe to run repeatedly.
3. Guardrails are evaluated: consent, quiet hours, weekly cap, and whether the member has already paid or renewed.
4. The job is queued to `messages` with exponential backoff and a provider-specific rate limit.
5. The provider webhook updates delivery status (sent, delivered, read, failed).
6. A payment event cancels pending reminders for that cycle.
7. Attribution: a payment within 7 days of a reminder is credited to "recovered by reminders".

### 6.9 Chatbot design (Release 2)
```
Inbound WhatsApp → webhook → thread resolver → Intent router
     ├ Known intent → deterministic handler (reads DB) → reply
     ├ FAQ / open question → retrieval (tenant knowledge, pgvector) → LLM → reply (grounded)
     └ Low confidence / sensitive → handoff to staff inbox
```
Tools exposed to the model are **read-only plus request-creation**. Prompts contain tenant data only for that tenant. Every conversation is logged, and prompt-injection defences apply (treat member text as data, never as instructions).

---

## 7. Non-functional requirements

| Category | Requirement |
|---|---|
| **Performance** | p95 API latency under 300 ms for standard reads; dashboard under 2 s; check-in search under 500 ms for 5,000 members |
| **Scalability** | Target 2,000 tenants and 500,000 members in year 1 on a single Postgres primary with read replica; horizontal API and worker scaling |
| **Availability** | 99.5% in MVP, 99.9% later; graceful degradation when WhatsApp or the payment gateway is down (queue and retry) |
| **Security** | TLS everywhere; encryption at rest; RLS tenant isolation; hashed OTP/refresh tokens; OWASP Top 10 controls; secrets in a manager; webhook signature checks; rate limiting; dependency scanning |
| **Privacy and compliance** | India DPDP Act: explicit consent, purpose limitation, data export and deletion on request, data stored in the India region; GST-compliant invoices; WhatsApp Business policy compliance (opt-in, templates, opt-out) |
| **Backups and DR** | Daily automated backups with point-in-time recovery; RPO 15 minutes, RTO 4 hours; quarterly restore test |
| **Auditability** | Immutable audit log for money and permission changes |
| **Usability** | Mobile-first; tap targets 44 px or larger; WCAG AA contrast; English and Hindi UI in MVP (i18n-ready from day one) |
| **Browser/device support** | Latest Chrome, Safari, Edge; Android Chrome on 2 GB RAM devices |
| **Observability** | Error tracking, tracing, queue-depth and failed-job alerts, per-tenant message cost metrics |
| **Testing** | Unit tests for money and membership date logic (100% of rules), integration tests with Postgres/Redis (Testcontainers), contract tests for webhooks, Playwright E2E for core flows |

---

## 8. UI/UX direction
- Light dashboard, dark navy sidebar, orange accent (`#FF6B2C`); status colours are reserved for meaning only (green = paid/active, amber = expiring, red = overdue).
- Typography: Poppins/Plus Jakarta Sans for headings, Inter for body and tables, Noto Sans Devanagari for Hindi.
- Design tokens (CSS variables) drive per-gym branding and dark mode.
- Front-desk screen: huge search, photo, status banner and one primary action.
- Bottom navigation on mobile: Home, Members, Check-in, Dues, Inbox.

---

## 9. Success metrics

| Type | Metric | Target |
|---|---|---|
| Product | Renewal rate uplift (pilot gyms) | +10 pp |
| Product | Dues recovered via reminders | ₹10,000+ per gym per month |
| Engagement | Weekly active staff users per gym | 80%+ |
| Efficiency | Median check-in time | Under 5 s |
| Activation | Gym live within 30 min of signup | 60%+ |
| Business | Trial-to-paid conversion | 25%+ |
| Business | Monthly logo churn | Under 3% |
| Reliability | Reminder delivery success | 97%+ |
| Quality | Money-related defects in production | 0 critical |

---

## 10. Pricing and packaging (to validate)
| Plan | Indicative price / month | Limits |
|---|---|---|
| Starter | ₹799-999 | 1 branch, up to ~150 members, reminders, billing |
| Growth | ₹1,799-2,499 | Up to ~500 members, chatbot, leads, trainer module |
| Pro | ₹3,999+ | Multi-branch, integrations, priority support |

WhatsApp usage is billed as prepaid message packs. Payment-gateway fees are passed through.

---

## 11. Delivery plan (MVP, 12 weeks)

| Weeks | Milestone | Deliverables |
|---|---|---|
| 1-2 | Discovery and design | 10 owner interviews, wireframes, design tokens, schema v1, repo, CI/CD, environments |
| 3-5 | Core | Auth/OTP, tenants/RLS, roles, members, plans, memberships, attendance, CSV import |
| 6-7 | Money | Invoices, payments, receipts (PDF), payment links, webhooks, dues tracker |
| 8-9 | Automation | Reminder engine, templates, WhatsApp integration, delivery webhooks, guardrails |
| 10 | Insight | Dashboard, caching, daily digest |
| 11-12 | Pilot and hardening | 3-5 pilot gyms, bug fixing, load test, security review, backup/restore drill |

Suggested team: 1 product/founder, 2 full-stack TypeScript engineers, 1 designer (part-time), 1 QA (part-time).

---

## 12. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| WhatsApp template rejection or policy change | Core feature blocked | Submit templates early; keep SMS/email fallback; use an official BSP only |
| Message spam leading to a number ban | Loss of channel | Consent, quiet hours, weekly caps, quality-rating monitoring |
| Money or date-logic bugs | Trust loss | Pure, heavily tested domain functions; append-only payments; audit log |
| Tenant data leak | Severe | RLS plus API checks, automated cross-tenant tests, penetration test before launch |
| Low owner tech literacy | Slow adoption | CSV import, guided setup, Hindi UI, video onboarding, WhatsApp support |
| Scope creep | Late MVP | Written non-goals; a feature enters only through the release plan |
| Competitors and free tools | Churn | Focus on collections ROI, speed and regional languages |
| Gateway or provider downtime | Failed sends | Queues, retries, dead-letter queue, status page |

---

## 13. Open questions
1. Which BSP is cheapest and most reliable for Indian traffic (Gupshup, Twilio, 360dialog, or Meta Cloud API direct)?
2. Razorpay or Cashfree: fees, UPI AutoPay support and onboarding speed for small merchants?
3. Will gyms pay for biometric integration, or is QR enough in the MVP?
4. Should the first release support multiple branches, or defer to Release 2?
5. How should a pro-rated upgrade be defined (by days remaining or by sessions)?
6. Hosting region and data residency choices for DPDP compliance.

---

## 14. Glossary
- **BSP:** WhatsApp Business Solution Provider
- **RLS:** PostgreSQL Row-Level Security
- **PWA:** Progressive Web App
- **RPO/RTO:** Recovery Point / Recovery Time Objective
- **DPDP:** Digital Personal Data Protection Act, 2023 (India)
- **ARPU / CAC:** Average revenue per user / Customer acquisition cost
