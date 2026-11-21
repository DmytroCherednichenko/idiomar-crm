# IDIOMAR — UI Wireframe Specifications

**Module 1:** Teacher Dashboard — Lessons & Pedagogic View: [wireframes-lessons.html](file:///Users/dima/Documents/Programming/idiomar-crm/docs/design/wireframes/wireframes-lessons.html)  
**Module 2:** Teacher Dashboard — Students Roster & Billing View: [wireframes-students.html](file:///Users/dima/Documents/Programming/idiomar-crm/docs/design/wireframes/wireframes-students.html)  
**Design System Reference:** [brand-guidelines.md](file:///Users/dima/Documents/Programming/idiomar-crm/docs/design/brand/brand-guidelines.md) | [branding-sheet.html](file:///Users/dima/Documents/Programming/idiomar-crm/docs/design/brand/branding-sheet.html)

---

## 1. Global Navigation Header (Teacher UI)

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [Logo] idiom[ar]        Home   Calendar   Students   [ Lessons ]          [🇺🇸 EN ▾]  [🔔]  [Avatar (G)] │
│                                                      ──────────                                        │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Components Breakdown:
1. **Brand Identity (Left):**
   * Directional diamond logo emblem SVG (`#FF7A30` top flame, `#000000` base facets).
   * Wordmark: **`idiom`** in Obsidian Black (`#000000`) + **`ar`** in Valencia Flame (`#FF7A30`). Eliminates redundancy while preserving domain identity.
2. **Main Navigation Bar (Middle):**
   * **`Home`**, **`Calendar`**, **`Students`**, and **`Lessons`**.
   * **Active State (`Lessons`):** Underlined with a crisp Valencia Flame (`#FF7A30`) stripe along the bottom border, paired with bold typography (orange dot removed).
3. **Utility Controls (Right):**
   * **Language Selector (`[🇺🇸 English (US) ▾]`):** Dropdown initially supporting **`English (US)`** and **`Español (LatAm)`** (German removed).
   * **Notification Center (`[🔔]`):** Bell icon with a subtle Valencia Flame (`#FF7A30`) unread badge for homework submissions.
   * **Teacher Profile Avatar (Prof. Camila Rossi):** Circular profile image with a Google OAuth badge indicating verified login status.

---

## 2. Sub-Header Toolbar (Filter & Action Bar)

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│  STUDENT                    SECTION                                                                    │
│  [ [SM] Sofia Morales ▾ ]   [ [📖 Lessons]  |  [🗂️ Vocabulary] ]                     [+ Schedule Lesson]│
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

1. **Student Context Dropdown:**
   * Shows only the student's avatar/initials (`[SM]`) and name (`Sofia Morales`).
   * Eliminates credit/balance counters to avoid visual noise; financial tracking is handled independently within the Students & Billing view.
2. **Segmented Section Toggle:**
   * Two-state switch between **`Lessons`** (selected view) and **`Vocabulary`**.
3. **Primary Action:**
   * **`+ Schedule Lesson`** CTA button styled in Valencia Flame (`#FF7A30`).

---

## 3. Lesson Card Anatomy & Status Lifecycle

Cards prioritize pedagogical clarity—lesson content, homework progress, and direct class access—without cluttering the view with financial transaction numbers or reference IDs.

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [Scheduled]  Tomorrow, Nov 22, 2026 • 15:00 - 16:00 (ART • 60 min)                 [ Join Class ]      │
│                                                                                                        │
│ Lesson #13: Subjunctive in Adverbial Clauses & Patagonia Travel Scenarios          [ Scheduled ]       │
│ Targeting conditional triggers: "a menos que", "para que", and "en caso de".                           │
│                                                                                                        │
│ ┌────────────────────────────────────────────────────────────────────────────┐                         │
│ │ 📝 Homework: Write 250-word essay on travel plans...   [Pending Submission] │                         │
│ └────────────────────────────────────────────────────────────────────────────┘                         │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Simplified Status Lifecycle:
* **Future Lessons:** Marked strictly as **`Scheduled`** (never marked as "paid" or "deducted").
* **Past Lessons:** Display one of three clean states:
  1. **`Waiting for payment`:** Amber status highlighting recent completed classes awaiting payment (no spinners, intrusive reference codes, or clutter).
  2. **`Confirmed`:** Reassuring green badge (`✓ Confirmed`) confirming settled sessions.
  3. **`Canceled`:** Muted neutral status for sessions canceled by tutor or student.

* **Unified Action Button:** All completed lessons feature the consistent pedagogical button **`[Recap & Words]`**, enabling the teacher to review summaries and flashcards at any time regardless of payment status. (Payments update to "Confirmed" automatically once settled).

---

## 4. Students Roster & Billing View (Module 2)

**File Reference:** [wireframes-students.html](file:///Users/dima/Documents/Programming/idiomar-crm/docs/design/wireframes/wireframes-students.html)

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [Logo] idiom[ar]        Home   Calendar   [ Students ]   Lessons          [🇺🇸 EN ▾]  [🔔]  [Avatar (G)] │
│                                           ────────────                                                 │
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ [ 🔍 Search students...          ]   [ Active (3) | Archived (1) ]               [ + Add New Student ]│
├────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ STUDENT & LANGUAGE              BALANCE               LEVEL                     ACTIONS                │
│ [SM] Sofia Morales              [● 3 classes left]    [ B2 ]     [ Schedule Class ]   [ ... ]          │
│ [LW] Lucas Weber                [● 0 classes left]    [ A2 ]     [ Schedule Class ]   [ ... ]          │
│ [ER] Elena Rostova              [● 8 classes left]    [ C1 ]     [ Schedule Class ]   [ ... ]          │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Components Breakdown:
1. **Toolbar Controls:**
   * **Real-Time Search Bar:** Filters student roster instantly by name or language.
   * **Segmented Status Filter:** Toggles between **`Active (3)`** and **`Archived (1)`** students to eliminate roster clutter without deleting database records.
   * **`+ Add New Student` CTA:** Primary modal trigger to register a student email, baseline CEFR proficiency, and agreed legacy hourly rate.
2. **Student Roster Columns:**
   * **Avatar & Name:** Visual avatar circle (photo or initials in Oxford Slate), display name, and mentorship start date.
   * **Class Balance Indicator:** Live balance from the financial ledger (`3 classes left` in emerald, `0 classes left · Needs renewal` in amber).
   * **CEFR Level Badge:** Common European Framework proficiency badge (`A2`, `B1`, `B2`, `C1`).
   * **Schedule Button:** Quick trigger to book next lesson in the tutor calendar.
   * **Three-Dots Menu (`...`):** Secondary contextual actions:
     * **`Change Price`**: Allows updating the agreed hourly lesson rate (`hourly_rate` in `student_profiles`).
     * **`Archive Student` / `Unarchive Student`**: Toggles `status = 'archived' | 'active'` without impacting payment ledgers or lesson records.

---

## 5. Slide-Out Student Detail Side Panel (Drawer)

Clicking any student row opens a right-side slide-over panel displaying their comprehensive dossier without navigating away from the roster:

```
┌──────────────────────────────────────────────────────────────┐
│  STUDENT DOSSIER                   [Active Student]   [ ✕ ] │
├──────────────────────────────────────────────────────────────┤
│  [SM]  Sofia Morales                                         │
│        sofia.morales@gmail.com                               │
│        [CEFR: B2] • Spanish for Business & Travel            │
├──────────────────────────────────────────────────────────────┤
│  PEDAGOGICAL NEEDS & LEARNING GOALS                          │
│  "Preparing for DELE B2 exam in November. Focus on advanced  │
│  subjunctive in adverbial clauses, formal email writing..." │
├──────────────────────────────────────────────────────────────┤
│  AGREED LESSON PRICE            CURRENT BALANCE              │
│  $24.00 USD / hour              3 lessons prepaid            │
├──────────────────────────────────────────────────────────────┤
│  [ + Log New Payment (Credit Lessons) ]                      │
├──────────────────────────────────────────────────────────────┤
│  PAYMENT HISTORY LEDGER                     Total: $432.00   │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ +$192.00 USD  [8 Lessons]                 [✓ Verified] │  │
│  │ Nov 05, 2026 • PayPal Transfer                         │  │
│  ├────────────────────────────────────────────────────────┤  │
│  │ +$144.00 USD  [6 Lessons]                 [✓ Verified] │  │
│  │ Oct 08, 2026 • Wise Transfer                           │  │
│  ├────────────────────────────────────────────────────────┤  │
│  │ +$96.00 USD   [4 Lessons]                 [✓ Verified] │  │
│  │ Sep 10, 2026 • Cash Handshake                          │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

* **Dossier & Goals:** Displays full student pedagogical objectives (`learning_goals`).
* **Agreed Rate:** Shows agreed legacy pricing protected per student.
* **Log Payment CTA:** One-click drawer form to record off-platform cash/PayPal/Wise transfers and credit 1, 4, 8, or 10 lessons immediately to the student balance.
* **Payment History Ledger:** Chronological audit trail showing payment date, transferred amount, credited lesson count, transfer channel, and verification status.

