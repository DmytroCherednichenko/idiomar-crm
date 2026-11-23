# IDIOMAR (idiom.ar) - Architecture & Database Diagrams

This directory contains the visual specifications for the **IDIOMAR** (idiom.ar) system (Proyecto Final de Graduación).

---

## 1. Entity-Relationship Diagram (ERD)
File: [01-erd.mmd](file:///Users/dima/Documents/Programming/IDIOMAR/docs/diagrams/01-erd.mmd)

```mermaid
erDiagram
    USERS ||--o| STUDENT_PROFILES : "has"
    USERS ||--o| TUTOR_PROFILES : "has"
    TUTOR_PROFILES ||--o{ STUDENT_PROFILES : "mentors"
    
    STUDENT_PROFILES ||--o{ LESSONS : "attends"
    TUTOR_PROFILES ||--o{ LESSONS : "conducts"
    
    STUDENT_PROFILES ||--o{ PAYMENTS : "submits"
    TUTOR_PROFILES ||--o{ PAYMENTS : "verifies"

    LESSONS ||--o{ HOMEWORK : "assigns"
    LESSONS ||--o{ VOCABULARY_ITEMS : "introduces"
    STUDENT_PROFILES ||--o{ VOCABULARY_ITEMS : "learns"

    USERS {
        uuid id PK
        string email UK
        string password_hash "nullable for Google OAuth"
        string google_id UK "nullable, Google user ID"
        string auth_provider "local | google"
        string full_name
        string role "tutor | student"
        string timezone "IANA timezone e.g. America/Argentina/Buenos_Aires"
        timestamp created_at
    }

    STUDENT_PROFILES {
        uuid user_id PK, FK
        uuid tutor_id FK
        string target_language
        string cefr_level "A1 to C2"
        decimal hourly_rate "Agreed rate per lesson (supports legacy pricing)"
        int default_payment_lessons "Default batch count e.g. 8 or 10"
        string currency "USD, EUR, etc."
        string status "active | archived"
    }

    TUTOR_PROFILES {
        uuid user_id PK, FK
        text payment_instructions "e.g. PayPal email, bank IBAN, payment notes"
        text google_refresh_token "nullable, encrypted OAuth token for GCal"
        string google_calendar_id "defaults to primary"
    }

    PAYMENTS {
        uuid id PK
        uuid student_id FK
        uuid verified_by_tutor_id FK "nullable until verified"
        decimal amount
        string currency
        int lessons_credited "e.g. 1, 5, 8, 10"
        string payment_method "PayPal | Wise | Bank Transfer | Cash"
        string status "pending_verification | verified | rejected"
        text student_note "e.g. PayPal transaction reference"
        timestamp requested_at
        timestamp verified_at
    }

    LESSONS {
        uuid id PK
        uuid tutor_id FK
        uuid student_id FK
        timestamp start_time
        int duration_minutes
        string status "scheduled | completed | canceled"
        string meeting_url "Google Meet / Zoom external URL"
        string google_event_id "nullable, Google Calendar event identifier"
        text lesson_summary
    }

    HOMEWORK {
        uuid id PK
        uuid lesson_id FK
        uuid student_id FK
        text title
        text instructions
        date due_date
        string status "pending | submitted | reviewed"
        text student_submission "Text or links to Google Docs/Drive"
        text teacher_feedback
    }

    VOCABULARY_ITEMS {
        uuid id PK
        uuid student_id FK
        uuid lesson_id FK "nullable if logged independently"
        string term
        string translation
        text context_sentence
        string category "e.g. grammar, travel, business"
        string mastery_status "new | learning | mastered"
        timestamp created_at
    }
```

---

## 2. C4 Level 2: Container Diagram
File: [02-c4-level2-containers.mmd](file:///Users/dima/Documents/Programming/IDIOMAR/docs/diagrams/02-c4-level2-containers.mmd)

```mermaid
flowchart TD
    classDef person fill:#08427b,stroke:#052e56,color:#ffffff,stroke-width:2px;
    classDef container fill:#1168bd,stroke:#0b4884,color:#ffffff,stroke-width:2px;
    classDef database fill:#1168bd,stroke:#0b4884,color:#ffffff,stroke-width:2px;
    classDef external fill:#888888,stroke:#555555,color:#ffffff,stroke-width:2px;
    classDef boundary fill:#f9f9f9,stroke:#999999,stroke-width:2px,stroke-dasharray: 5 5;

    Tutor["fa:fa-user Tutor\n[Person]\nIndependent language instructor managing classes, payments, and students"]:::person
    Student["fa:fa-user Student\n[Person]\nLanguage learner attending lessons, reviewing vocabulary, and reporting payments"]:::person

    subgraph ExternalServices["External Systems & Channels"]
        GoogleAuth["Google Identity Platform\n[External: Google OAuth 2.0 / OpenID Connect]\nAuthenticates students via Google Sign-In and issues ID tokens"]:::external
        GoogleCalendar["Google Calendar API\n[External: Google Calendar REST v3]\nDirectly pushes scheduled lessons, syncs updates, and generates Meet rooms"]:::external
        EmailService["Email Service Provider\n[External: Resend]\nSends booking confirmations, reminders, and notifications"]:::external
        MeetingPlatform["Video Conferencing\n[External: Google Meet / Zoom]\nThird-party video call rooms launched via external URLs"]:::external
        PaymentChannels["Payment Channels\n[External: PayPal / Wise / Banks]\nOff-platform monetary transfers executed directly by users"]:::external
    end

    subgraph IDIOMARSystem["System Boundary: IDIOMAR (idiom.ar)"]
        WebApp["Web Application (SPA)\n[Container: Angular / TypeScript]\nDelivers responsive UI for dashboards, glossary decks, calendar scheduling, and payment management"]:::container

        ApiServer["API & Application Server\n[Container: Node.js / NestJS or Express, TypeScript]\nHandles business logic, auth/JWT verification, Google Calendar synchronization, payment reconciliations, and vocabulary search"]:::container

        Database[("Relational Database\n[Container: PostgreSQL]\nStores users, profiles, lessons, homework, vocabulary cards, and payment logs")]:::database
    end

    Tutor -->|"Views dashboard, schedules classes, confirms payments, and logs vocabulary via HTTPS"| WebApp
    Student -->|"Views schedule and glossary deck; may optionally report a payment via HTTPS"| WebApp

    WebApp -->|"Makes API requests\n[HTTPS / JSON / REST]"| ApiServer
    ApiServer -->|"Reads and writes domain data\n[TCP / SQL via Prisma or Drizzle ORM]"| Database
    ApiServer -->|"Dispatches transactional emails\n[HTTPS / REST API]"| EmailService
    ApiServer -->|"Directly pushes lesson events & Meet links\n[HTTPS / REST API v3]"| GoogleCalendar
    ApiServer -->|"Verifies Google ID tokens & OAuth tokens\n[HTTPS / OAuth 2.0]"| GoogleAuth

    WebApp <-->|"OAuth 2.0 / Google Sign-In flow\n[HTTPS]"| GoogleAuth
    WebApp -.->|"Redirects / opens video room URL\n[HTTPS]"| MeetingPlatform
    Student -.->|"Transfers funds off-platform\n[PayPal, Wise, Cash]"| PaymentChannels
    Tutor -.->|"Checks balance / verifies incoming transfer"| PaymentChannels
```

---

## 3. C4 Level 3: Component Diagram (Modular Monolith Backend)
File: [03-c4-level3-components.mmd](file:///Users/dima/Documents/Programming/IDIOMAR/docs/diagrams/03-c4-level3-components.mmd)

```mermaid
flowchart TD
    classDef client fill:#1168bd,stroke:#0b4884,color:#ffffff,stroke-width:2px;
    classDef module fill:#2d74da,stroke:#1a4d99,color:#ffffff,stroke-width:2px;
    classDef shared fill:#4a8fe7,stroke:#2b6bb8,color:#ffffff,stroke-width:2px;
    classDef db fill:#1168bd,stroke:#0b4884,color:#ffffff,stroke-width:2px;
    classDef external fill:#888888,stroke:#555555,color:#ffffff,stroke-width:2px;
    classDef boundary fill:#f8f9fa,stroke:#707070,stroke-width:2px,stroke-dasharray: 5 5;

    Client["Web App Frontend (Angular SPA)\n[Container]"]:::client

    subgraph ModularMonolith["API Application Server (Modular Monolith) [Container: Node.js / NestJS or Express]"]
        
        ApiGateway["API Routing & Controllers Layer\n(Route Handlers / Auth Middleware)"]:::shared
        
        subgraph CoreModules["Bounded Business Modules"]
            AuthMod["Identity & Access Module\n• Auth & JWT verification\n• Google OAuth token verification\n• User & Profile management\n• Role-based guards (Tutor vs Student)"]:::module
            
            PedagogicMod["Pedagogic Module\n• Glossary deck & Flashcards\n• Vocabulary mastery tracking\n• Homework assignment & review"]:::module
            
            SchedulingMod["Scheduling Module\n• Booking & Rescheduling\n• Timezone normalization\n• Google Calendar sync & Meet integration"]:::module
            
            BillingMod["Billing & Analytics Module\n• Payment confirmation handshake\n• Lesson credit ledger calculation\n• Tutor revenue & KPI dashboard"]:::module
        end

        subgraph SharedInfrastructure["Shared Infrastructure"]
            NotificationMod["Notification Service\n• Dispatches transactional emails\n• Formats event templates"]:::shared
            DataAccess["Data Access Layer\n(Prisma / Drizzle ORM)\n• Repositories & Connection Pool"]:::shared
        end
    end

    Database[("PostgreSQL Database\n[Container]")]:::db
    EmailService["Email Service (Resend)\n[External System]"]:::external
    GoogleCalendar["Google Calendar API (v3)\n[External System]"]:::external
    GoogleAuth["Google Identity Platform\n[External System]"]:::external

    Client -->|"HTTPS / REST API Requests"| ApiGateway

    ApiGateway --> AuthMod
    ApiGateway --> PedagogicMod
    ApiGateway --> SchedulingMod
    ApiGateway --> BillingMod

    SchedulingMod -.->|"Lesson completed event"| BillingMod
    BillingMod -.->|"Triggers receipt email"| NotificationMod
    SchedulingMod -.->|"Triggers booking email"| NotificationMod

    AuthMod --> DataAccess
    PedagogicMod --> DataAccess
    SchedulingMod --> DataAccess
    BillingMod --> DataAccess

    DataAccess -->|"SQL / TCP Connections"| Database

    NotificationMod -->|"HTTPS / REST"| EmailService
    SchedulingMod -->|"Direct event sync & Meet generation\n[HTTPS / OAuth 2.0]"| GoogleCalendar
    AuthMod -->|"Verifies ID tokens\n[HTTPS / OAuth 2.0]"| GoogleAuth
```

---

## 4. UML Sequence Diagram: Lesson Lifecycle (Scheduling & Completion)
File: [04-sequence-lesson-lifecycle.mmd](file:///Users/dima/Documents/Programming/IDIOMAR/docs/diagrams/04-sequence-lesson-lifecycle.mmd)

```mermaid
sequenceDiagram
    autonumber
    actor Tutor as Tutor (Browser)
    participant UI as Angular SPA (Frontend)
    participant API as API Gateway / Router
    participant Sched as Scheduling Module
    participant GCal as Google Calendar API
    participant Ped as Pedagogic Module
    participant Bill as Billing Module
    participant DB as PostgreSQL Database
    participant Notif as Notification Service
    actor Student as Student (Browser/Email)

    rect rgb(240, 248, 255)
        note over Tutor, Student: PHASE 1: Scheduling a Lesson
        Tutor->>UI: Selects Student, Date/Time, Duration & Meeting URL (or auto-generate)
        UI->>API: POST /api/lessons {student_id, start_time, duration, meeting_url}
        API->>Sched: scheduleLesson(dto)
        
        Sched->>DB: Check for schedule conflicts (time overlaps)
        DB-->>Sched: Conflict check OK (no overlaps)

        opt If tutor enabled Google Calendar sync
            Sched->>GCal: POST /events (sync event, set reminders & auto-create Meet room)
            GCal-->>Sched: 200 OK {id: google_event_id, hangoutLink: meeting_url}
        end
        
        Sched->>DB: INSERT INTO lessons (status = 'scheduled', meeting_url, google_event_id)
        DB-->>Sched: Lesson record created (id: uuid)
        
        Sched-)Notif: emit LessonScheduledEvent(lessonDetails)
        Notif--)Student: Send email notification with Google Meet/Zoom link
        
        Sched-->>API: 201 Created (LessonDTO)
        API-->>UI: 201 Created (JSON)
        UI-->>Tutor: Displays confirmed lesson on Calendar
    end

    rect rgb(245, 255, 245)
        note over Tutor, Student: PHASE 2: Completing Lesson & Post-Class Logging
        Tutor->>UI: Clicks "Complete Lesson" & fills recap, new vocab words, optional homework
        UI->>API: POST /api/lessons/{id}/complete {summary, vocab_items, homework}
        API->>Sched: completeLesson(id, completionData)
        
        Sched->>DB: BEGIN transaction, lock lesson, confirm it is scheduled
        Sched->>DB: UPDATE lessons SET status = 'completed', summary = ... WHERE id = ...
        DB-->>Sched: Lesson updated
        
        opt If new vocabulary items provided
            Sched->>Ped: recordVocabulary(student_id, lesson_id, vocab_items)
            Ped->>DB: INSERT INTO vocabulary_items (terms, translations, category)
            DB-->>Ped: Vocab entries persisted
        end
        
        opt If homework assigned
            Sched->>Ped: assignHomework(lesson_id, student_id, homeworkData)
            Ped->>DB: INSERT INTO homework (title, instructions, due_date)
            DB-->>Ped: Homework record persisted
        end

        Sched->>Bill: createCharge(lesson, effectiveRate, currency)
        Bill->>DB: INSERT one immutable charge for the completed billable lesson
        Bill->>DB: Settle oldest charges FIFO only when same-currency credit covers each charge in full
        DB-->>Bill: Returns paid or unpaid charge status and remaining unallocated credit
        Sched->>DB: COMMIT transaction
        
        Sched-)Notif: emit LessonSummaryReadyEvent(student_id, lesson_id)
        Notif--)Student: Sends email with lesson recap & new vocabulary to review
        
        Sched-->>API: 200 OK (CompletedLessonDTO)
        API-->>UI: 200 OK (JSON)
        UI-->>Tutor: Updates Dashboard (Hours taught counter incremented)
    end
```

---

## 5. UML Sequence Diagram: Homework Lifecycle
File: [06-sequence-homework-lifecycle.mmd](file:///Users/dima/Documents/Programming/IDIOMAR/docs/diagrams/06-sequence-homework-lifecycle.mmd)

```mermaid
sequenceDiagram
    autonumber
    actor Tutor as Tutor (Browser)
    participant UI as Angular SPA (Frontend)
    participant API as API Gateway / Router
    participant Ped as Pedagogic Module
    participant DB as PostgreSQL Database
    participant Notif as Notification Service
    actor Student as Student (Browser)

    rect rgb(240, 248, 255)
        note over Tutor, Student: PHASE 1: Assigning Homework
        Tutor->>UI: Fills title, instructions, due date (attached to lesson)
        UI->>API: POST /api/homework {lesson_id, student_id, title, instructions, due_date}
        API->>Ped: assignHomework(dto, tutorId)
        
        Ped->>DB: INSERT INTO homework (status = 'pending', created_at = NOW())
        DB-->>Ped: Homework record persisted (id: uuid)
        
        Ped-)Notif: emit HomeworkAssignedEvent(homeworkDetails)
        Notif--)Student: Sends email: "New homework assigned: [Title]" with due date
        
        Ped-->>API: 201 Created (HomeworkDTO)
        API-->>UI: 201 Created (JSON)
        UI-->>Tutor: Displays assignment in Lesson details
    end

    rect rgb(255, 250, 240)
        note over Tutor, Student: PHASE 2: Student Submission
        Student->>UI: Opens Homework tab, writes answer or pastes Google Doc/Drive link
        Student->>UI: Clicks "Submit Homework"
        UI->>API: PUT /api/homework/{id}/submit {student_submission}
        API->>Ped: submitHomework(id, studentId, student_submission)
        
        Ped->>DB: UPDATE homework SET student_submission = ..., status = 'submitted', submitted_at = NOW() WHERE id = ...
        DB-->>Ped: Homework updated
        
        Ped-)Notif: emit HomeworkSubmittedEvent(studentId, homeworkId)
        Notif--)Tutor: Dashboard alert & email: "Student submitted homework for review"
        
        Ped-->>API: 200 OK (HomeworkDTO)
        API-->>UI: 200 OK (JSON)
        UI-->>Student: Displays status "Submitted - Awaiting Review"
    end

    rect rgb(245, 255, 245)
        note over Tutor, Student: PHASE 3: Tutor Review & Feedback
        Tutor->>UI: Opens Dashboard -> Clicks "Pending Homework to Correct"
        Tutor->>UI: Reads submission, writes pedagogical feedback
        Tutor->>UI: Clicks "Complete Review"
        UI->>API: PATCH /api/homework/{id}/review {teacher_feedback}
        API->>Ped: reviewHomework(id, tutorId, teacher_feedback)
        
        Ped->>DB: UPDATE homework SET teacher_feedback = ..., status = 'reviewed', reviewed_at = NOW() WHERE id = ...
        DB-->>Ped: Homework updated
        
        Ped-)Notif: emit HomeworkReviewedEvent(studentId, homeworkId)
        Notif--)Student: Sends email: "Your homework was reviewed with teacher feedback"
        
        Ped-->>API: 200 OK (ReviewedHomeworkDTO)
        API-->>UI: 200 OK (JSON)
        UI-->>Tutor: Dashboard updates (Pending review counter decremented)
    end
```

---

## 6. UML Sequence Diagram: Lesson Charges & Payment Allocation

This agreed flow replaces the old payment-confirmation sequence. **Completing a billable lesson creates its charge; a payment does not create a charge.** Students may optionally report payment details, but the tutor can record a payment directly by entering the verified amount. Verified credit settles charges FIFO, and only when it can cover a charge in full. Any amount that cannot cover the oldest open charge remains as unallocated student credit. The ERD and data dictionary still describe the legacy lesson-credit model and have not yet been updated.

File: [07-sequence-charge-payment-allocation.mmd](./07-sequence-charge-payment-allocation.mmd)

```mermaid
sequenceDiagram
    autonumber
    actor Tutor as Tutor (Browser)
    participant UI as Angular SPA (Frontend)
    participant API as API Gateway / Router
    participant Sched as Scheduling Module
    participant Bill as Billing Module
    participant DB as PostgreSQL Database
    participant Notif as Notification Service
    actor Student as Student (Browser/Email)

    rect rgb(245, 255, 245)
        note over Tutor, Student: PHASE 1: Complete a Lesson and Create Its Charge
        Tutor->>UI: Marks lesson complete and submits recap
        UI->>API: POST /api/lessons/{id}/complete {summary, vocab_items, homework}
        API->>Sched: completeLesson(id, completionData)
        Sched->>DB: BEGIN transaction, lock lesson, confirm it is scheduled
        DB-->>Sched: Lesson is eligible for completion
        Sched->>DB: UPDATE lesson status = 'completed' and save recap
        Sched->>Bill: createCharge(lesson, effectiveRate, currency)
        Bill->>DB: INSERT charge (lesson_id, student_id, amount_due, currency)
        Note over Bill, DB: One immutable charge per completed billable lesson — snapshot the rate and currency
        Bill->>DB: Lock student billing record, settle oldest charges fully when existing same-currency credit covers each charge
        DB-->>Bill: Return settled charges and remaining unallocated credit
        Bill->>DB: COMMIT transaction
        Sched-->>API: 200 OK (CompletedLessonDTO, charge summary)
        API-->>UI: 200 OK (JSON)
        UI-->>Tutor: Shows lesson lifecycle and separate payment badge
        Sched-)Notif: emit LessonSummaryReadyEvent(studentId, lessonId)
        Notif--)Student: Sends lesson recap and new vocabulary
    end

    rect rgb(240, 248, 255)
        note over Student, Tutor: PHASE 2: Optional Student Report and Tutor-Recorded Payment
        Note over Student: Student sends money off-platform (e.g. PayPal, Wise, Bank)
        opt Student chooses to report payment details
            Student->>UI: Optionally enters amount, currency, method, and note
            UI->>API: POST /api/payments/reports {reportedAmount?, currency?, paymentMethod?, note?}
            API->>Bill: reportPayment(optionalDetails, studentId)
            Bill->>DB: Save optional report details (status = 'pending_verification')
            DB-->>Bill: Payment report created
            Bill-)Notif: emit PaymentReportedEvent(paymentReport)
            Notif--)Tutor: Alerts tutor to review the optional payment report
            Bill-->>API: 201 Created (PaymentReportDTO)
            API-->>UI: 201 Created (JSON)
            UI-->>Student: Displays report status
        end

        Tutor->>UI: Selects a student and opens a report or records a payment directly
        UI->>API: POST /api/students/{studentId}/payments {reportedPaymentId?, verifiedAmount, currency?, paymentMethod?, tutorNote?}
        API->>Bill: recordVerifiedPayment(studentId, tutorId, paymentDetails)
        Bill->>DB: BEGIN transaction, lock student billing record, payment, and relevant open charges
        alt Existing student report selected
            Bill->>DB: Mark report verified with verified_amount and tutor
        else Tutor records payment without a student report
            Bill->>DB: Create verified payment with verified_amount and tutor
        end
        Bill->>DB: Add verified amount to same-currency unallocated credit
        Bill->>DB: Settle oldest charges FIFO only when pooled credit covers each charge in full
        alt Credit covers the oldest open charge
            DB-->>Bill: Settle full charge and continue to the next oldest charge
        else Credit cannot cover the oldest open charge or no open charges remain
            DB-->>Bill: Retain remaining amount as unallocated student credit and leave any open charge unpaid
        end
        Bill->>DB: COMMIT transaction
        Bill-)Notif: emit PaymentRecordedEvent(paymentId, settledCharges, remainingCredit)
        Notif--)Student: Sends receipt, settled charges, and any remaining credit
        Bill-->>API: 200 OK (RecordedPaymentDTO)
        API-->>UI: 200 OK (JSON)
        UI-->>Tutor: Shows paid or unpaid charges, credit, and payment history
    end
```

### Implementation Notes

1. **Add a `charges` table.** Create one charge for each billable completed lesson, with a unique `lesson_id`, `student_id`, `amount_due`, `currency`, and creation timestamp. Store the final amount and currency at charge creation so later rate changes do not rewrite history.
2. **Add a `payment_allocations` table.** Link verified payments to charges with `payment_id`, `charge_id`, `amount_applied`, and `allocated_at`. Create allocation rows only when pooled same-currency credit can settle the full charge; the rows may draw from multiple payments, and one payment may settle multiple charges. Each committed allocation set must cover the charge in full, never leave it partially paid.
3. **Change `payments`.** Keep optional `reported_amount`, `payment_method`, and student note fields separate from the tutor-entered `verified_amount`. Require the verified amount to record a payment, default currency from the student's billing currency where appropriate, and record the tutor and verification time. Remove `lessons_credited`; it no longer describes the ledger. Unallocated credit is verified payment value not yet used to fully settle charges.
4. **Update lesson completion and payment recording.** In transactions, complete the lesson/create exactly one charge or record a verified payment, then apply same-currency credit FIFO. Make completion idempotent, enforce one charge per lesson, and serialize billing updates per student so concurrent operations cannot reuse the same credit.
5. **Revise billing queries and UI.** Show completed charges as paid or unpaid, plus the student's unallocated credit. Do not expose a partially paid charge state.
6. **Plan legacy data migration.** Existing payments record `lessons_credited`, and old lessons lack historical charge amounts. A reliable migration needs a chosen rate-history assumption or manual reconciliation; the current data cannot always reveal which old lessons a payment covered or what each cost.
7. **Define exceptional billing rules.** Decide how to handle canceled/late-canceled lessons, refunds, payment corrections, and charge corrections. Preserve an audit trail rather than silently editing or deleting financial history.

The existing data dictionary and ERD still describe the legacy credit-count model. Update them to match this agreed charge-and-allocation flow before implementation, including its migration and exceptional-billing rules.
