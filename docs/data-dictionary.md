# IDIOMAR - Database Data Dictionary

This document provides the complete data dictionary for the relational database schema of **IDIOMAR** (PostgreSQL). It defines table specifications, column data types, integrity constraints, and business domain rules.

---

## 1. Domain Enums & Value Sets

| Enum Type / Check Constraint | Allowed Values | Description |
| :--- | :--- | :--- |
| `user_role` | `'tutor'`, `'student'` | Role assigned to user for authorization guards |
| `auth_provider` | `'local'`, `'google'` | Authentication identity provider for user credentials |
| `cefr_level` | `'A1'`, `'A2'`, `'B1'`, `'B2'`, `'C1'`, `'C2'` | Common European Framework of Reference for Languages |
| `lesson_status` | `'scheduled'`, `'completed'`, `'cancelled'`, `'rescheduled'` | Lifecycle status of a booked lesson |
| `homework_status` | `'pending'`, `'submitted'`, `'reviewed'` | Progression status of an assigned homework |
| `payment_status` | `'pending_verification'`, `'verified'`, `'rejected'` | Handshake state between student payment and tutor review |
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
**Description:** Pedagogical and financial profile for each student, establishing their mentorship link to a specific tutor and their legacy pricing.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `user_id` | `UUID` | No | `PRIMARY KEY`, `FK -> users(id) ON DELETE CASCADE` | None | One-to-one relationship with `users` |
| `tutor_id` | `UUID` | No | `FK -> tutor_profiles(user_id) ON DELETE RESTRICT` | None | The independent tutor mentoring this student |
| `target_language` | `VARCHAR(50)` | No | None | None | Language the student is learning (e.g. Spanish, German) |
| `cefr_level` | `VARCHAR(5)` | Yes | `CHECK (cefr_level IN ('A1','A2','B1','B2','C1','C2'))` | `'A1'` | Assessed CEFR proficiency level |
| `hourly_rate` | `DECIMAL(10,2)` | No | `CHECK (hourly_rate >= 0)` | None | Agreed lesson price for this student (preserves legacy pricing) |
| `default_payment_lessons` | `INT` | No | `CHECK (default_payment_lessons > 0)` | `8` | Default batch size pre-selected on payment renewal modal |
| `currency` | `VARCHAR(3)` | No | ISO 4217 code | `'USD'` | Billing currency agreed with tutor |
| `learning_goals` | `TEXT` | Yes | None | `NULL` | Student's motivation, target exams, or notes |

---

### 2.4 Table: `lessons`
**Description:** Record of scheduled, completed, and cancelled language classes.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | No | `PRIMARY KEY` | `gen_random_uuid()` | Unique lesson identifier |
| `tutor_id` | `UUID` | No | `FK -> tutor_profiles(user_id) ON DELETE CASCADE` | None | Tutor conducting the session |
| `student_id` | `UUID` | No | `FK -> student_profiles(user_id) ON DELETE CASCADE` | None | Student attending the session |
| `start_time` | `TIMESTAMPTZ` | No | None | None | Session start timestamp stored in UTC |
| `duration_minutes` | `INT` | No | `CHECK (duration_minutes IN (30, 45, 60, 90, 120))` | `60` | Class duration in minutes |
| `status` | `VARCHAR(20)` | No | `CHECK (status IN ('scheduled','completed','cancelled','rescheduled'))` | `'scheduled'` | Current operational status |
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

### 2.7 Table: `payments`
**Description:** Verification ledger tracking off-platform payment confirmations and credited lesson quantities.

| Column Name | Data Type | Nullable | Constraints | Default | Description / Business Rules |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | No | `PRIMARY KEY` | `gen_random_uuid()` | Unique payment transaction record |
| `student_id` | `UUID` | No | `FK -> student_profiles(user_id) ON DELETE CASCADE` | None | Student reporting the payment |
| `verified_by_tutor_id` | `UUID` | Yes | `FK -> tutor_profiles(user_id) ON DELETE SET NULL` | `NULL` | Tutor who verified and approved the payment |
| `amount` | `DECIMAL(10,2)` | No | `CHECK (amount > 0)` | None | Total monetary amount transferred |
| `currency` | `VARCHAR(3)` | No | ISO 4217 code | `'USD'` | Currency of transaction |
| `lessons_credited` | `INT` | No | `CHECK (lessons_credited > 0)` | `8` | Number of lessons purchased with this transaction |
| `payment_method` | `VARCHAR(50)` | No | None | `'PayPal'` | Transfer channel (e.g. `'PayPal'`, `'Wise'`, `'Bank Transfer'`, `'Cash'`) |
| `status` | `VARCHAR(25)` | No | `CHECK (status IN ('pending_verification','verified','rejected'))` | `'pending_verification'` | Handshake verification state |
| `student_note` | `VARCHAR(255)` | Yes | None | `NULL` | Transaction reference code or note provided by student |
| `tutor_note` | `VARCHAR(255)` | Yes | None | `NULL` | Rejection reason or internal remark from tutor |
| `requested_at` | `TIMESTAMPTZ` | No | None | `CURRENT_TIMESTAMP` | When student requested verification |
| `verified_at` | `TIMESTAMPTZ` | Yes | None | `NULL` | Timestamp when tutor approved or rejected payment |

---

## 3. Financial Ledger Formula (Lesson Balance)

A student's live balance of available lessons is computed dynamically without relying on mutable counters:

```sql
SELECT 
    s.user_id AS student_id,
    COALESCE(SUM(p.lessons_credited), 0) - 
    COALESCE((SELECT COUNT(*) FROM lessons l WHERE l.student_id = s.user_id AND l.status = 'completed'), 0) 
    AS available_lesson_balance
FROM student_profiles s
LEFT JOIN payments p ON p.student_id = s.user_id AND p.status = 'verified'
GROUP BY s.user_id;
```
* **Positive Value (`> 0`):** Number of prepaid classes available for future booking.
* **Zero (`= 0`):** Up to date; no debt, no prepaid credit.
* **Negative Value (`< 0`):** Number of completed lessons awaiting student payment.
