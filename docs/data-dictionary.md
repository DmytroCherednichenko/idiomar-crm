# IDIOMAR - Database Data Dictionary

This document provides the complete data dictionary for the relational database schema of **IDIOMAR** (PostgreSQL). It defines table specifications, column data types, integrity constraints, and business domain rules.

---

## 1. Domain Enums & Value Sets

| Enum Type / Check Constraint | Allowed Values | Description |
| :--- | :--- | :--- |
| `user_role` | `'tutor'`, `'student'` | Role assigned to user for authorization guards |
| `auth_provider` | `'local'`, `'google'` | Authentication identity provider for user credentials |
| `student_status` | `'active'`, `'archived'` | Lifecycle state of a student profile (active learner vs. archived) |
| `cefr_level` | `'A1'`, `'A2'`, `'B1'`, `'B2'`, `'C1'`, `'C2'` | Common European Framework of Reference for Languages |
| `lesson_status` | `'scheduled'`, `'completed'`, `'canceled'` | Lifecycle status of a booked lesson |
| `homework_status` | `'pending'`, `'submitted'`, `'reviewed'` | Progression status of an assigned homework |
| `payment_report_status` | `'pending'`, `'recorded'`, `'dismissed'` | State of an optional student-submitted payment report; confirmed payments are stored separately |
| `mastery_status` | `'new'`, `'learning'`, `'mastered'` | Learning retention level in the Glossary Deck |

---

## 2. Table Specifications

### 2.1 Table: `users`
**Description:** Central identity table storing authentication credentials, basic contact information, role, and timezone preferences. Supports both local password authentication and Google OAuth sign-in.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | No | `PRIMARY KEY` | `gen_random_uuid()` | Unique identifier for each user |
| `email` | `VARCHAR(255)` | No | `UNIQUE`, valid email | None | Login credential; must be unique across system |
| `password_hash` | `VARCHAR(255)` | Yes | None | `NULL` | Bcrypt or Argon2 hashed password (nullable for students/users logging in via Google) |
| `google_id` | `VARCHAR(255)` | Yes | `UNIQUE` | `NULL` | Google's unique user ID (`sub`); null for password/local accounts |
| `auth_provider` | `VARCHAR(20)` | No | `CHECK (auth_provider IN ('local', 'google'))` | `'local'` | Authentication method / identity provider (`'local'` or `'google'`) |
| `full_name` | `VARCHAR(150)` | No | None | None | Display name of the user |
| `role` | `VARCHAR(20)` | No | `CHECK (role IN ('tutor', 'student'))` | `'student'` | Distinguishes between tutor and student |
| `timezone` | `VARCHAR(50)` | No | Valid IANA timezone | `'UTC'` | Timezone identifier (e.g. `'America/Argentina/Buenos_Aires'`) used for calendar rendering |
| `created_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | Account creation timestamp |
| `updated_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | Last profile update timestamp |

---

### 2.2 Table: `tutor_profiles`
**Description:** Profile extension storing settings, default rate, off-platform payment instructions, and Google Calendar sync tokens for language tutors.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `user_id` | `UUID` | No | `PRIMARY KEY`, `FK -> users(id) ON DELETE CASCADE` | None | One-to-one relationship with `users` |
| `bio` | `TEXT` | Yes | None | `NULL` | Brief pedagogical bio or introduction |
| `default_hourly_rate` | `DECIMAL(10,2)` | Yes | `CHECK (default_hourly_rate >= 0)` | `NULL` | Baseline rate used when onboarding new students |
| `default_currency` | `VARCHAR(3)` | No | ISO 4217 code (e.g. `'USD'`) | `'USD'` | Default billing currency |
| `payment_instructions`| `TEXT` | Yes | None | `NULL` | Details shown to students (e.g. PayPal email, bank IBAN, payment notes) |
| `google_refresh_token`| `TEXT` | Yes | None | `NULL` | Encrypted OAuth 2.0 refresh token for Google Calendar API event synchronization |
| `google_calendar_id`  | `VARCHAR(255)` | Yes | None | `'primary'` | Target Google Calendar ID (defaults to tutor's primary calendar) |

---

### 2.3 Table: `student_profiles`
**Description:** Pedagogical and financial profile for each student, establishing their mentorship link to a specific tutor and agreed lesson pricing.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `user_id` | `UUID` | No | `PRIMARY KEY`, `FK -> users(id) ON DELETE CASCADE` | None | One-to-one relationship with `users` |
| `tutor_id` | `UUID` | No | `FK -> tutor_profiles(user_id) ON DELETE RESTRICT` | None | The independent tutor mentoring this student |
| `target_language` | `VARCHAR(50)` | No | None | None | Language the student is learning (e.g. Spanish, German) |
| `cefr_level` | `VARCHAR(5)` | Yes | `CHECK (cefr_level IN ('A1','A2','B1','B2','C1','C2'))` | `'A1'` | Assessed CEFR proficiency level |
| `hourly_rate` | `DECIMAL(10,2)` | No | `CHECK (hourly_rate >= 0)` | None | Agreed hourly rate used to calculate new lesson charges |
| `currency` | `VARCHAR(3)` | No | ISO 4217 code | `'USD'` | Billing currency agreed with tutor |
| `learning_goals` | `TEXT` | Yes | None | `NULL` | Student's motivation, target exams, or notes |
| `status` | `VARCHAR(20)` | No | `CHECK (status IN ('active', 'archived'))` | `'active'` | Lifecycle state; allows archiving past students to declutter UI while preserving past lessons & payments |

---

### 2.4 Table: `lessons`
**Description:** Record of scheduled, completed, and canceled language classes.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | No | `PRIMARY KEY`, `UNIQUE (id, student_id)` | `gen_random_uuid()` | Unique lesson identifier; composite unique key supports charge ownership integrity |
| `tutor_id` | `UUID` | No | `FK -> tutor_profiles(user_id) ON DELETE CASCADE` | None | Tutor conducting the session |
| `student_id` | `UUID` | No | `FK -> student_profiles(user_id) ON DELETE CASCADE` | None | Student attending the session |
| `start_time` | `TIMESTAMPTZ` | No | None | None | Session start timestamp stored in UTC |
| `duration_minutes` | `INT` | No | `CHECK (duration_minutes IN (30, 45, 60, 90, 120))` | `60` | Class duration in minutes |
| `status` | `VARCHAR(20)` | No | `CHECK (status IN ('scheduled','completed','canceled'))` | `'scheduled'` | Current operational status |
| `meeting_url` | `VARCHAR(500)` | Yes | Valid URL | `NULL` | External video call room (Google Meet generated via Calendar API, or Zoom link) |
| `google_event_id` | `VARCHAR(255)` | Yes | None | `NULL` | Google Calendar event ID for direct synchronization, updates, and cancellations |
| `lesson_summary` | `TEXT` | Yes | None | `NULL` | Pedagogical notes and recap logged by tutor after class |
| `created_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | Audit creation timestamp |

---

### 2.5 Table: `homework`
**Description:** Homework assignments created by the tutor, submitted by the student, and reviewed with pedagogical feedback.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | No | `PRIMARY KEY` | `gen_random_uuid()` | Unique homework identifier |
| `lesson_id` | `UUID` | Yes | `FK -> lessons(id) ON DELETE SET NULL` | `NULL` | Optional link to the specific lesson where assigned |
| `student_id` | `UUID` | No | `FK -> student_profiles(user_id) ON DELETE CASCADE` | None | Student responsible for the assignment |
| `title` | `VARCHAR(200)` | No | None | None | Title or subject of the assignment |
| `instructions` | `TEXT` | No | None | None | Markdown instructions provided by the tutor |
| `due_date` | `DATE` | No | None | None | Target completion date |
| `status` | `VARCHAR(20)` | No | `CHECK (status IN ('pending','submitted','reviewed'))` | `'pending'` | Homework lifecycle state |
| `student_submission` | `TEXT` | Yes | None | `NULL` | Student's answer or external link (Google Docs/Drive) |
| `teacher_feedback` | `TEXT` | Yes | None | `NULL` | Pedagogical comments and corrections from tutor |
| `submitted_at` | `TIMESTAMPTZ` | Yes | None | `NULL` | Timestamp when student submitted work |
| `reviewed_at` | `TIMESTAMPTZ` | Yes | None | `NULL` | Timestamp when tutor finalized feedback |
| `created_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | Assignment creation timestamp |

---

### 2.6 Table: `vocabulary_items`
**Description:** Dedicated vocabulary entries belonging to a student's personal Glossary Deck for interactive spaced repetition and categorization.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | No | `PRIMARY KEY` | `gen_random_uuid()` | Unique vocabulary entry identifier |
| `student_id` | `UUID` | No | `FK -> student_profiles(user_id) ON DELETE CASCADE` | None | Student who owns this glossary card |
| `lesson_id` | `UUID` | Yes | `FK -> lessons(id) ON DELETE SET NULL` | `NULL` | Lesson where the word was introduced (if applicable) |
| `term` | `VARCHAR(150)` | No | None | None | Target language word, phrasal verb, or expression |
| `translation` | `VARCHAR(255)` | No | None | None | Native language translation or target language definition |
| `context_sentence`| `TEXT` | Yes | None | `NULL` | Pedagogical example sentence demonstrating usage |
| `category` | `VARCHAR(50)` | Yes | None | `'General'` | Classification tag (e.g. `'Grammar'`, `'Business'`, `'Travel'`) |
| `mastery_status` | `VARCHAR(20)` | No | `CHECK (mastery_status IN ('new','learning','mastered'))` | `'new'` | Retention level for flashcard practice |
| `review_count` | `INT` | No | `CHECK (review_count >= 0)` | `0` | Times reviewed in the Glossary Deck |
| `created_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | When word was added to deck |

---

### 2.7 Table: `charges`
**Description:** Immutable charge for one completed billable lesson. The amount and currency are snapshotted when the lesson is completed so later rate changes do not alter billing history.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | No | `PRIMARY KEY`, `UNIQUE (id, student_id, currency)` | `gen_random_uuid()` | Unique charge identifier; composite unique key supports allocation integrity |
| `lesson_id` | `UUID` | No | `UNIQUE`, composite `FK -> lessons(id, student_id) ON DELETE RESTRICT` | None | At most one charge per billable lesson |
| `student_id` | `UUID` | No | `FK -> student_profiles(user_id) ON DELETE RESTRICT` | None | Student responsible for the charge; composite FK ensures this matches the lesson |
| `amount_due` | `DECIMAL(10,2)` | No | `CHECK (amount_due > 0)` | None | Final charge amount, snapshotted at completion |
| `currency` | `VARCHAR(3)` | No | ISO 4217 code | None | Currency of this charge; must match allocations |
| `charged_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | FIFO order for charges; use `id` as a stable tie-breaker |

Charge payment state is derived from allocations: a charge is paid when allocation amounts sum to `amount_due`; otherwise it is unpaid. Allocation transactions must never commit a partially paid charge.

---

### 2.8 Table: `payment_reports`
**Description:** Optional details a student chooses to submit about an off-platform transfer. A report is not a confirmed payment and never changes credit by itself. A tutor can also record a confirmed payment without any report.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | No | `PRIMARY KEY`, `UNIQUE (id, student_id)` | `gen_random_uuid()` | Unique report identifier; composite unique key supports payment ownership integrity |
| `student_id` | `UUID` | No | `FK -> student_profiles(user_id) ON DELETE RESTRICT` | None | Student who submitted the report |
| `reported_amount` | `DECIMAL(10,2)` | Yes | `CHECK (reported_amount IS NULL OR reported_amount > 0)` | `NULL` | Amount the student says they sent; may be omitted |
| `reported_currency` | `VARCHAR(3)` | Yes | ISO 4217 code | `NULL` | Reported currency; may be omitted |
| `payment_method` | `VARCHAR(50)` | Yes | None | `NULL` | Reported transfer channel; may be omitted |
| `student_note` | `VARCHAR(255)` | Yes | None | `NULL` | Optional reference or note |
| `status` | `VARCHAR(20)` | No | `CHECK (status IN ('pending','recorded','dismissed'))` | `'pending'` | `recorded` when linked to a confirmed payment; `dismissed` when not accepted |
| `created_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | When the student submitted the report |
| `resolved_at` | `TIMESTAMPTZ` | Yes | None | `NULL` | When the tutor recorded or dismissed the report |
| `resolved_by_tutor_id` | `UUID` | Yes | `FK -> tutor_profiles(user_id) ON DELETE SET NULL` | `NULL` | Tutor who resolved the report |
| `tutor_note` | `VARCHAR(255)` | Yes | None | `NULL` | Optional internal note when dismissing or resolving the report |

Do not create a report when the student chooses not to submit one. If all report fields are empty, treat the interaction as no report.

---

### 2.9 Table: `payments`
**Description:** Ledger of funds the tutor has confirmed were received. Every row is verified; student reports are optional and stored separately.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | No | `PRIMARY KEY`, `UNIQUE (id, student_id, currency)` | `gen_random_uuid()` | Unique confirmed payment identifier; composite unique key supports allocation integrity |
| `student_id` | `UUID` | No | `FK -> student_profiles(user_id) ON DELETE RESTRICT` | None | Student whose payment was received |
| `payment_report_id` | `UUID` | Yes | `UNIQUE`, composite `FK -> payment_reports(id, student_id)` | `NULL` | Optional report being verified; null for direct tutor entry |
| `verified_amount` | `DECIMAL(10,2)` | No | `CHECK (verified_amount > 0)` | None | Amount the tutor confirms was received |
| `currency` | `VARCHAR(3)` | No | ISO 4217 code | None | Confirmed currency; default from student billing currency in the UI/service |
| `payment_method` | `VARCHAR(50)` | Yes | None | `NULL` | Confirmed transfer channel, if known |
| `verified_by_tutor_id` | `UUID` | No | `FK -> tutor_profiles(user_id) ON DELETE RESTRICT` | None | Tutor who recorded and confirmed receipt |
| `verified_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | When the tutor confirmed receipt |
| `tutor_note` | `VARCHAR(255)` | Yes | None | `NULL` | Optional note for a directly recorded or verified payment |

Payments are immutable financial facts; corrections should be represented by auditable adjustment records rather than silently changing the verified amount.

---

### 2.10 Table: `payment_allocations`
**Description:** Records which verified funds settle which lesson charges. The many-to-many relationship allows multiple payments to contribute to one fully paid charge and one payment to settle multiple charges.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `payment_id` | `UUID` | No | `PK`, `FK -> payments(id) ON DELETE RESTRICT` | None | Confirmed payment providing these funds |
| `charge_id` | `UUID` | No | `PK`, `FK -> charges(id) ON DELETE RESTRICT` | None | Charge settled by these funds |
| `student_id` | `UUID` | No | Composite FK with `payment_id` and `charge_id` | None | Ensures payment and charge belong to the same student |
| `currency` | `VARCHAR(3)` | No | Composite FK with `payment_id` and `charge_id` | None | Ensures payment and charge currencies match |
| `amount_applied` | `DECIMAL(10,2)` | No | `CHECK (amount_applied > 0)` | None | Portion of this payment applied to this charge |
| `allocated_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | When the allocation was recorded |

The composite primary key is `(payment_id, charge_id)`. Composite foreign keys to `(payment_id, student_id, currency)` and `(charge_id, student_id, currency)` ensure allocations never cross students or currencies. Create allocations only when pooled credit can fully cover the oldest open charge; in the same transaction, record all contributing payment portions so committed allocations for the charge sum to its full amount.

---

### 2.11 Table: `student_billing_accounts`
**Description:** Mutable, transactionally maintained summary of a student's unallocated credit for one currency. Payment and allocation rows remain the auditable source of truth; this table provides a fast balance read.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `student_id` | `UUID` | No | `PK`, `FK -> student_profiles(user_id) ON DELETE CASCADE` | None | Student owning this billing account |
| `currency` | `VARCHAR(3)` | No | `PK`, ISO 4217 code | None | Credit is kept separately for each currency |
| `unallocated_credit` | `DECIMAL(10,2)` | No | `CHECK (unallocated_credit >= 0)` | `0` | Verified funds not yet allocated to fully settle charges |
| `updated_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | Last balance update |

The composite primary key is `(student_id, currency)`. Create an account row when a student first incurs a charge or receives payment in that currency. Update this balance in the same transaction as payment insertion and charge allocations, locking the account row to prevent concurrent updates from spending the same credit twice. Do not combine this value with outstanding charges into one net balance.

---

## 3. Financial Ledger and Cached Credit

`student_billing_accounts.unallocated_credit` is the mutable operational balance, maintained atomically with the immutable payment and allocation records. Reconcile it against those records with:

```sql
WITH payment_totals AS (
    SELECT student_id, currency, SUM(verified_amount) AS total_verified
    FROM payments
    GROUP BY student_id, currency
),
allocation_totals AS (
    SELECT student_id, currency, SUM(amount_applied) AS total_allocated
    FROM payment_allocations
    GROUP BY student_id, currency
)
SELECT
    account.student_id,
    account.currency,
    account.unallocated_credit AS stored_credit,
    COALESCE(payment.total_verified, 0) - COALESCE(allocation.total_allocated, 0)
        AS calculated_credit
FROM student_billing_accounts account
LEFT JOIN payment_totals payment
    ON payment.student_id = account.student_id AND payment.currency = account.currency
LEFT JOIN allocation_totals allocation
    ON allocation.student_id = account.student_id AND allocation.currency = account.currency;
```

The stored and calculated values should match. A charge is **paid** when its allocations sum to `amount_due`; otherwise it is **unpaid**. Allocation runs FIFO by `charged_at` (then `id`) and only settles a charge when the available same-currency credit can cover it in full. If credit is insufficient for the oldest open charge, stop and retain all remaining credit as unallocated.
