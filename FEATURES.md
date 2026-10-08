# Feature Reference

Every feature listed here is built and running in production. Organized by module, with the surfaces each one appears on.

**W** = Web · **M** = Mobile · **P** = Student/Parent portal

---

## Students

| Feature | Surfaces | Notes |
|---|---|---|
| Student records | W M | Full profile, guardians, contacts, medical, documents |
| New admission | W M | Auto-generated admission number, photo capture |
| Bulk import | W M | Excel with dry-run preview before applying |
| Bulk update | W | Edit many records at once from a spreadsheet |
| Bulk photo upload | W M | Excel + ZIP, matched by admission number |
| Promotion | W | Move a cohort to the next class and session |
| Sibling groups | W | Admin-approved; enables one-login-many-children |
| Student leaves | W M P | Apply, review, approve/reject |
| Transfer certificate | W M | Customizable layout and HTML templates, register numbering by book and serial |
| Password management | W M | Issue, reset, and bulk credential export; a password the user chose themselves is never shown to the office |
| Student info | W M | House, height, weight, blood group, eyesight — entered class by class, with Excel import/export |
| Roll numbers | W M | Alphabetical per class and section, on demand |
| Information sheet | W | Printable six-per-page student cards with photo |
| Photo export | W M | ZIP named by admission number, plus a missing-photos list |

Sibling grouping deserves a note: it drives the parent profile switcher, so it's admin-approved rather than inferred from a shared phone number. Two unrelated families can share a number through a typo, and auto-linking them would expose one family's child to another.

---

## Attendance

| Feature | Surfaces | Notes |
|---|---|---|
| Student daily attendance | W M | Present/Absent/Medical/Leave/Half-day |
| Staff attendance | W M | Same statuses, separate register |
| Offline marking | M | Queues locally, syncs when signal returns |
| Bulk upload | W M | Register-matrix Excel, dry-run first |
| Monthly register | W M P | Per-student month view with totals |
| Absence notifications | — | Automatic push to parents on save |
| Class-teacher scoping | W M | A teacher sees only their own section |
| School calendar | W M P | Holidays, vacations, exams, PTMs; holidays pre-fill the register |
| Back-dating control | W M | Marking a past date needs its own permission |

---

## Fees

The largest module.

| Feature | Surfaces | Notes |
|---|---|---|
| Fee structure setup | W | Per class, session, and fee type |
| Installments | W | Due dates with optional labels |
| Fee collection | W M | Partial payments, multiple modes |
| Discounts | W M | Percentage or fixed; stackable; per-installment |
| Discount on fines | W M | Waive the fine while collecting the fee |
| Late fines | W | Configurable rules, late-admission policy |
| Previous session balance | W | Carry-forward, with bulk upload |
| Dues report | W M | Filterable, exportable |
| Fee Statement export | W M | Register-style Bill/Deposit/Dues by due date |
| Defaulter list | W | Configurable threshold |
| Receipts | W M | Browser print and on-device PDF; edit in place under the same number; reprints keep the dues as on the day of issue |
| Exemptions | W M | Removing a fee from a student is remembered, so re-assignment never re-bills it |
| Concession import | W M | A whole school's discounts from one spreadsheet |
| Fee setup export | W M | Structure, installments, assignments and ledger to Excel or JSON |
| Fee visibility toggle | W | Per-tenant: hide fees from students entirely |

### UPI payments

| Step | Surface | Behaviour |
|---|---|---|
| Initiate | P | Deep link to a UPI app, or QR for another device |
| Submit proof | P | Upload payment screenshot, optional UTR note |
| Track | P | Incomplete / Pending / Approved / Rejected, with reason |
| Approve | W M | Creates the receipt atomically |

Approval creates the fee transaction and flips the request in one transaction, guarded so a double-click cannot produce two receipts. UPI UI hides entirely when the school hasn't configured a payment address.

---

## Examinations

| Feature | Surfaces | Notes |
|---|---|---|
| Terms and exams | W M | Per session, with soft-delete and restore |
| Exam-class setup | W M | Subjects, max marks, passing marks |
| Marks entry | W M | AB / ML / NA statuses distinct from zero; papers split into parts, each with its own status; grade-only subjects |
| Term formulas | W M | Per class: best-of, average, sum, rescale, round — defined as data |
| Grade schemes | W M | Custom bands per class or section; pick-from-a-list scales for activities |
| Co-scholastic | W M | Report-card blocks, remark bank, per-student fields (homework, PTM, attendance) |
| Teacher remarks | W M | Per subject and per class, with Excel import/export |
| Result publishing | W M | Per class or per section, frozen snapshot, notifies students |
| Ranks | W M | Within section or class; 1-2-2-4 or 1-2-2-3; medical-leave and new-admission students in or out |
| Tabulation sheet | W M | Full class grid with totals and rank, print and Excel |
| Report cards | W M | School-owned HTML templates; filled or hollow stars; printed on the phone as PDFs |
| Term setup file | W M | Export a whole term's setup, import it into another term or school |
| Syllabus & datesheet | W M P | Published independently of exams |
| Fee-defaulter gate | W P | Optionally lock results until dues cleared |
| Student electives | W M | Per-student subject selection, tick-grid Excel import |

The absent/medical distinction matters more than it looks: a student absent for one paper shouldn't have a zero averaged into their percentage, and the policy for how each status affects numerator and denominator is configurable per school.

---

## Transport

| Feature | Surfaces | Notes |
|---|---|---|
| Routes and vehicles | W M | With driver details |
| Fee tiers | W M | Amount-based, independent of route |
| Student assignment | W M | Per session |
| Fee generation | W | Monthly, bi-monthly, quarterly clubbing |
| Fare revision | W M | Mid-session, with preview and revert |
| Per-student adjustment | W M | Raise or lower one student's fare |
| Waivers | W M | Down to individual months |
| Opt-out | W M | With residue handling on rejoin |

Fare changes floor at the amount already paid, so a reduction never creates a negative balance or an implied refund.

---

## Academics

| Feature | Surfaces | Notes |
|---|---|---|
| Classes and sections | W M | Per-section class teacher |
| Subjects | W M | Class and section mapping |
| Timetable | W M P | Periods, teachers, breaks |
| Homework | W M P | With attachments |
| Homework submission | P | File upload or mark-as-done |
| Study notes | W M P | Shared materials with attachments |

---

## Accounting

| Feature | Surfaces | Notes |
|---|---|---|
| Accounts | W | Cash, bank, and cheque accounts |
| Account heads | W | Income and expense categories |
| Income | W | Manual entries plus automatic fee posting |
| Expenses | W | With vendor and payment tracking |
| Cheque management | W | Pending → cleared/declined, undo a mistaken clear |
| Statements | W | Account-wise financial reports |

---

## Communication

| Feature | Surfaces | Notes |
|---|---|---|
| Direct messages | W M P | Staff ↔ staff, staff ↔ student/parent |
| Broadcasts | W M | Whole school, staff, students |
| Class broadcast | W M | Specific classes and sections |
| Attachments | W M P | Documents and images |
| Push notifications | M P | Firebase, identity-aware |
| Notification inbox | W M P | Categorized, with deep links |

---

## Front Desk

| Feature | Surfaces | Notes |
|---|---|---|
| Gate pass | W M | Digital approval or physical signature mode |
| Approval chain | W M | Push to approvers, approve in-app |
| Printing | W M | Thermal and A4 layouts |
| Staff out-pass | W M | Request from own phone, approve, mark out and back in |

---

## Staff

| Feature | Surfaces | Notes |
|---|---|---|
| Employee records | W M | Full profile, documents, photo |
| Staff attendance | W M | Daily marking and monthly register |
| Leave management | W M | Apply, approve, track balance |
| Tasks | W M | Assign, track, pass on, submit |
| Employee ↔ child switching | M | Staff with children at the school |

---

## Administration

| Feature | Surfaces | Notes |
|---|---|---|
| Roles and permissions | W M | 73 features, 231 keys; export/import between schools; a read-only role for app-store reviewers |
| Permission matrix | W M | Visual grid editor |
| Academic sessions | W | Create, activate, manage |
| Session repair tools | W | Fix registered sessions from admission dates |
| General settings | W | Branding, contact, UPI, portal toggles |
| Change log | W M | Automatic before/after of every data change, with a readable summary per event |
| Audit logs | W M | Filterable, hash-chained |
| Dashboard | W M | Cards chosen per role, attendance progress by section, a "today" panel |
| Notification switches | W | Per school, all off or one kind at a time |
| App usage | W M | Who has the app installed and opened, who doesn't |
| Forced update | W | Per-school minimum app version |
| Backup and restore | W | Per-tenant, scheduled and manual |
| Super admin console | W | Provision and manage school tenants |

---

## Student & Parent Portal

Available on both mobile and web:

- Dashboard with attendance percentage and pending dues
- Attendance record, month by month
- Fee dues, payment history, and UPI payment
- Exam results with grades and teacher remarks
- Homework with submission
- Study notes and timetable
- Leave application with status tracking
- Messaging with staff
- Notification inbox
- Profile and password management
- Sibling switching for parents with several children

---

## Cross-cutting

**Offline-first** where it matters — attendance, fee collection, leave requests and tasks queue locally and sync with idempotency protection.

**Excel and PDF exports** throughout, styled to match the layouts schools already use on paper, with one consistent file-naming rule; printed by the browser or made into a PDF on the phone — no print server.

**Old computers supported** — every screen renders on Chrome / Edge 109, the last versions available on Windows 7, still common in school offices.

**India-specific handling** — timezone-correct dates everywhere (a date is the school's date, not the server's), Indian numbering in currency display, UPI payment integration, and an April–March academic session model.

**White-label** — each school gets its own branded mobile app and web instance from a single codebase.
