# IDIOMAR - Technology Stack Justification

**Project:** CRM para Tutores de Idiomas Independientes IDIOMAR 
**Authors:** Axel Ramos & Dmytro Cherednichenko  
**Document Type:** Technical & Architectural Evaluation (Proyecto Final de Graduación)

---

## 1. Executive Summary & Constraints

The architectural choices for **IDIOMAR** were determined by balancing academic rigor, software engineering best practices, and the concrete operational constraints defined in the project charter (*Acta de Constitución del Proyecto*):

* **Human Resource Budget:** 240 total person-hours (120 hours per developer, ~7.5 hours/week).
* **Delivery Deadline:** March 5, 2027 (Backend completion: Dec 2026; Frontend integration: Feb 2027).
* **Key Performance Indicators (KPIs):**
  * System availability / uptime: **$\ge 99\%$** during pilot testing.
  * API and page response times: **$< 1.5$ seconds** on primary views.
  * Stability: **$100\%$** coverage of critical business flows (roles, scheduling, vocabulary, payment verification).

---

## 2. Frontend Architecture: Angular (TypeScript)

### 2.1 Why Angular for a Tutor CRM?
While consumer websites often opt for React/Next.js for search engine optimization (SEO), **IDIOMAR is an authenticated internal web application (CRM/Dashboard)**. Angular provides distinct advantages for this domain:

1. **Robust Form Handling (`ReactiveFormsModule`):**
   * Language tutoring CRMs are form-intensive (recording post-class summaries, submitting homework, configuring custom hourly rates, dynamic vocabulary inputs, and vacation payment adjustments).
   * Angular's reactive forms provide built-in validation, dynamic form arrays, and type-safe control models out of the box, reducing third-party dependency fragmentation.
2. **Standardized Role-Based Navigation (`CanActivate` Guards):**
   * The platform requires strict isolation between **Tutor** views (revenue metrics, review queues, student pricing) and **Student** views (personal deck, homework submissions).
   * Angular provides native route guards that enforce role-based access control (RBAC) cleanly at the client router level.
3. **Team Velocity & Developer Synergy:**
   * Axel has prior production experience with Angular. In a 2-person project with a strict 240-hour ceiling, leveraging proven developer familiarity dramatically lowers technical risk and accelerates sprint delivery.
4. **Modern Architecture (Signals & Standalone Components):**
   * Modern Angular (v17+) eliminates the historical overhead of `NgModules` in favor of lightweight standalone components and fine-grained reactive signals, delivering high UI performance and sub-second rendering.

---

## 3. Backend Architecture: Node.js / NestJS (Modular Monolith)

### 3.1 Why NestJS & TypeScript?
1. **Architectural Symmetry with Angular:**
   * NestJS was intentionally designed around the same design patterns as Angular (controllers, providers, modules, dependency injection, and decorators).
   * This architectural harmony allows both Axel and Dmytro to seamlessly read, review, and contribute to both ends of the codebase using consistent TypeScript syntax and conventions.
2. **Modular Monolith Pattern (Bounded Contexts):**
   * Rather than introducing the operational complexity, network latency, and distributed transaction pitfalls of **microservices**, the system is structured as a **Modular Monolith**.
   * Business logic is segregated into discrete modules (`IdentityModule`, `PedagogicModule`, `SchedulingModule`, `BillingModule`, `NotificationModule`).
   * Evaluators in academic thesis defenses strongly favor this approach as it demonstrates **Domain-Driven Design (DDD)** and high cohesion without overengineering.
3. **Single Deployable Unit:**
   * Eliminates complex multi-container orchestration in production, making it trivial to achieve the **99% uptime KPI** while keeping infrastructure hosting costs at zero or minimal student tier pricing.

---

## 4. Persistence Layer: PostgreSQL & Modern ORM

### 4.1 Database Engine: PostgreSQL
1. **Relational Integrity & ACID Guarantees:**
   * Financial transactions (payment verifications and class credits) and scheduling reservations demand strict ACID compliance to prevent race conditions (e.g. double bookings or erroneous credit balances).
2. **Timezone Accuracy (`TIMESTAMPTZ`):**
   * Independent language tutors frequently teach students living in different countries and timezones. PostgreSQL’s native handling of UTC timestamps combined with IANA timezone strings guarantees accurate calendar calculations without DST (Daylight Saving Time) drift.
3. **Check Constraints & Enums:**
   * Business constraints (e.g. positive credit amounts, CEFR level validation `A1-C2`, lesson durations of 30–120 minutes) are enforced at the database schema level, preventing data corruption regardless of API client bugs.

### 4.2 ORM Layer: Prisma or Drizzle ORM
* **End-to-End Type Safety:** Database schema definitions automatically generate TypeScript types that are shared directly with API controllers and DTOs (Data Transfer Objects).
* **Automated Migrations:** Database schema evolution is fully version-controlled, reproducible across development environments, and verifiable in CI pipelines.

---

## 5. Scope Boundary & External Integrations

To honor the project charter's *Exclusiones (Fuera de Alcance V1)* and complete development within 240 person-hours, third-party integrations were strategically bounded:

| Component | Architecture Decision | Justification |
| :--- | :--- | :--- |
| **Authentication** | **Hybrid Local & Google OAuth 2.0** | Tutors authenticate via local credentials (email/password); students can sign in with either local credentials or Google OAuth (via Google Identity Services). Minimizes onboarding friction for language learners while preserving instructor independence. |
| **Payments** | **Manual Verification Handshake** (Off-platform PayPal / Wise / Cash) | Eliminates PCI-DSS compliance overhead, banking gateway approval delays, merchant fees, and country-specific KYC requirements. Perfectly models how independent tutors operate in practice. |
| **Video Calls** | **Hybrid Generated & Custom URLs** (Google Meet / Zoom URL) | Leverages Google Calendar API to auto-generate native Google Meet rooms on event creation, with fallback support for manual video URLs (Zoom, Skype). Eliminates WebRTC implementation overhead while automating meeting link setup. |
| **Calendar Sync** | **Google Calendar API (Direct Sync)** | Direct REST integration with Google Calendar API (v3) using tutor OAuth 2.0 authorization. Completely eliminates the severe 8–24 hour polling lag of static iCalendar (.ics) feeds. Enables instant push synchronization for class bookings, rescheduling, cancellations, and native calendar notifications. |
| **Email Service** | **Transactional API (Resend)** | Modern cloud email service tailored for TypeScript/Node.js ecosystems with first-class SDK support. Dispatches asynchronous notifications for scheduling confirmations and payment receipts with sub-100ms dispatch latency while remaining 100% cost-free on the 3,000 emails/month free tier. |

---

## 6. Summary of Architectural Alignment

```
┌─────────────────────────────────────────────────────────────┐
│                     IDIOMAR (idiom.ar)                      │
├──────────────────────────────┬──────────────────────────────┤
│ Frontend                     │ Angular (TypeScript, SPA)    │
│ Backend                      │ Node.js / NestJS (Mod-Mono)  │
│ Language                     │ TypeScript (End-to-End)      │
│ Database                     │ PostgreSQL (with ORM)        │
│ Authentication               │ Local (JWT) & Google OAuth   │
│ Architecture Pattern         │ Modular Monolith / DDD       │
│ Calendar Integration         │ Google Calendar API (REST v3)│
│ Video Hosting                │ Meet (Auto) / Custom URL     │
│ Email Notifications          │ Resend (REST API)            │
│ Payment Processing           │ Manual Ledger Verification   │
│ Target Infrastructure        │ Dockerized PaaS / VPS        │
└──────────────────────────────┴──────────────────────────────┘
```

This stack satisfies every academic requirement for software quality, security, and architectural separation of concerns, while directly protecting the delivery schedule and KPI commitments of the degree project.
