# AY Voice

## 📣 Overview
**AY Voice** is a secure, enterprise‑grade bulk‑communication platform built for businesses in Bangladesh. It consolidates **SMS (masking & non‑masking), voice call, and email marketing** into a single, intuitive dashboard, enabling marketers to launch multi‑channel campaigns at scale while maintaining strict budget control through a wallet‑based billing system.

---

## 🏗️ Architecture

```mermaid
flowchart TD
    subgraph Client
        UI["Web UI (Next.js 16)"]
    end
    subgraph API
        Hono["Hono (Bun)"] -->|HTTP| Auth["Auth Layer"]
        Auth -->|JWT| DB[("PostgreSQL 16")]
        Auth -->|Cache| Redis[("Redis 7")]
        Hono -->|Queue| BullMQ["BullMQ Workers"]
        BullMQ -->|SMS| Awaj["AwajDigital"]
        BullMQ -->|Voice| Awaj
        BullMQ -->|Email| Brevo["Brevo (Sendinblue)"]
    end
    UI -->|REST| Hono
    UI -->|WebSocket| Redis
    classDef external fill:#f9f,stroke:#333,stroke-width:2px;
    class Awaj,Brevo external;
```

*The diagram above is a high‑level view of the data flow between the front‑end, API, queues, and external providers.*

---

## 🛠️ Tech Stack
| Layer | Technology |
|---|---|
| **Runtime** | Bun (≥ 1.3) |
| **Frontend** | Next.js 16 (App Router) • React 19 • Tailwind CSS 4 |
| **Backend** | Hono 4 (running on Bun) |
| **Database** | PostgreSQL 16 + Prisma 7 ORM |
| **Queue** | Redis 7 + BullMQ 6 |
| **Auth** | JWT (jose) • HttpOnly cookies • bcrypt |
| **Payments** | ZiniPay |
| **Voice** | AwajDigital |
| **Email** | Brevo (Sendinblue) • Resend (transactional) |
| **CAPTCHA** | Cloudflare Turnstile |
| **Observability** | Sentry (error tracking) • Pino (structured logging) |
| **Deployment** | Docker • Docker‑Compose • PM2 • Nginx |
| **CI/CD** | GitHub Actions |

---

## 📁 Project Structure (sanitized)
```
ayvoice/
├── api/                # Hono API server (Bun) – routes, middleware, core services
│   ├── prisma/         # Prisma schema, migrations, seed data
│   └── src/            # Business logic, providers, workers
├── web/                # Next.js 16 front‑end – pages, components, hooks
│   ├── app/            # App Router pages
│   └── components/     # Reusable UI components
├── shared/             # Shared TypeScript utilities (workspace package)
├── docs/               # Documentation, screenshots, case‑study PDFs
├── docker-compose.yml  # Orchestration for Postgres, Redis, API, Web
└── .github/workflows/  # CI / Deploy pipelines
```

---

## ✨ Key Features (excerpt)
- **Unified Campaign Management** – Create, schedule, and monitor SMS, voice, and email campaigns from a single dashboard.
- **Wallet‑Based Billing** – Pre‑pay credits; each send deducts the exact cost, with automatic refunds on failures.
- **Admin Approval Workflow** – Email campaigns require admin review before dispatch, ensuring compliance.
- **Robust Queueing** – BullMQ batches sends (500 contacts per batch) and retries with exponential back‑off.
- **Provider‑Agnostic Architecture** – Adapter pattern allows effortless swapping of SMS/voice providers.
- **Real‑Time Analytics** – Delivery rates, open/click metrics, and financial reporting.
- **Security Hardened** – HttpOnly JWT cookies, refresh‑token rotation, bcrypt hashing, Cloudflare Turnstile, rate‑limiting, and comprehensive webhook validation.

---

## 👤 Demo Account (HR testing)
| Email | Password |
|---|---|
| `hr-demo@ayvoice.test` | `AyVoiceDemo123!` |

Use the above credentials to explore the UI, run test campaigns, and view analytics. All operations run against a **demo database**; no real funds are affected.

---

## 📸 Project Showcase & Interface Preview

![AY Voice Dashboard Preview](./AyVoice.png)

> **Case Study Highlight:** A high-volume enterprise customer scaled outreach by **35%** with automated DLR reconciliation and masking SMS routing, cutting cost per engagement while maintaining a 99.4% queue processing success rate.

---

## 🔐 Security & Architecture Highlights
- **JWT & Refresh‑Token Rotation** – Detects token theft via family‑based revocation.
- **bcrypt (cost 12)** – Secure password storage.
- **Turnstile CAPTCHA** – Mitigates automated abuse on authentication endpoints.
- **Rate‑Limiting** – Nginx + application‑level limits + fail2ban.
- **Webhook Verification** – HMAC‑SHA256 signatures for Brevo and delivery receipts.
- **Idempotent Processing** – Event hashing prevents duplicate handling.
- **Atomic Quotas** – Row-level lock quota reservation to prevent multi-worker over-dispatching.

---

## 📦 Deployment (Docker)
```bash
# Build and start all services
docker compose up -d
# Services exposed:
#   - PostgreSQL :5432
#   - Redis      :6379
#   - API        :5050
#   - Web UI     :3000
```

Production containers are built using multi‑stage Dockerfiles (`Dockerfile.api` and `Dockerfile.web`) and managed by **PM2** for zero‑downtime restarts.

---

## 📧 Requesting Full Source Code
For a full security audit, please email **security@ayvoice.test** with a brief justification. After verification, the complete repository will be shared through a secure channel.

---

## 📄 License
Private – All rights reserved.
