# School ERP — Open-Source K-12 School Management Platform

A self-hostable, white-label ERP for K-12 schools. One deployment serves many schools, each with its own isolated database, branding, and mobile app.

Built to replace the paper registers, spreadsheets, and WhatsApp groups that most schools still run on — with something a school can own outright rather than rent.

**Status:** Production. Running live for real schools with ~900 students, ~70 staff, and daily attendance, fee collection, examinations, and parent communication.

---

## Why this exists

School ERP software is a solved problem for well-funded institutions and an unsolved one for everyone else. Commercial platforms charge per-student annual fees, hold the data, and put customization behind support tickets. Schools that can't afford them run on paper: attendance in registers, fees in ledgers, marks in spreadsheets, announcements in WhatsApp groups.

The costs are quiet but real. A parent can't find out their child was absent until the monthly meeting. A fee receipt exists on one carbon copy. Report card season means a week of manual arithmetic. Nobody can answer "how many students have unpaid transport fees for August" without an afternoon of work.

This project exists so that a school with no IT budget can self-host a system that handles all of it, brand it as their own, and never pay per student.

---

## What it does

**68 permission-gated features across 16 modules.** Every screen listed here is built and in production use.

### Students
Admissions and full student records, promotion between sessions, sibling-group management, document and photo handling, bulk import/update from Excel, and transfer certificate generation with a customizable form layout.

### Attendance
Daily marking for students and staff, offline-capable on mobile (marks queue on the phone and sync when signal returns), monthly registers, bulk upload via a register-matrix spreadsheet, and automatic absence notifications to parents.

### Fees
The largest module. Fee structures per class and session, installments with due dates, discounts (percentage or fixed, stackable, targeted at specific installments or at fines), late fines, previous-session carry-forward balances, and collection with partial payments and multiple payment modes.

Reporting includes dues reports, defaulter lists, and a register-style **Fee Statement** export that lays out Bill / Deposit / Dues by due-date across class bands — the format schools actually use on paper.

**UPI payments:** parents pay from the mobile app via deep link or QR, upload the payment screenshot, and the office approves it — creating the receipt atomically. Every submission has a visible status; a rejection states why.

### Examinations
Terms, exams, per-class subject setup, marks entry (offline-safe), absent/medical statuses that don't count as zero, configurable grade schemes, co-scholastic grading, teacher remarks at subject and class level, tabulation sheets, and report cards. Results publish per class, with an optional fee-defaulter gate that locks results until dues are cleared.

### Transport
Routes, vehicles, fee tiers, and per-student assignment. Monthly fee generation with configurable clubbing (monthly, bi-monthly, quarterly), mid-session fare revision with a revert path, per-student fare adjustment, waivers down to individual months, and opt-out handling.

### Accounting
Accounts and account heads, income and expense tracking, cheque lifecycle management (pending → cleared/declined, with reversal), vendor records, and financial statements. Fee collection posts into accounting automatically with double-entry integrity.

### Academics
Classes and sections with per-section class teachers, subjects and electives, timetables, homework with submission tracking, and study notes.

### Communication
Direct messaging between staff, students, and parents, plus broadcasts to the whole school, all staff, all students, or specific classes and sections. Push notifications via Firebase Cloud Messaging reach the right device even when several family members share one phone.

### Front Desk
Gate-pass workflow with a digital approval chain or a physical signature mode, printable in thermal and A4 formats.

### Staff
Employee records, attendance, leave requests and approvals, role assignment, and an internal task system for delegating work between staff.

### Administration
Role-based access control with 68 features and granular actions, academic session management, audit logging with a tamper-evident hash chain, database backup and restore, and a super-admin console for provisioning new schools.

---

## Who uses it

The platform serves four distinct audiences from one codebase:

- **Administrators and office staff** — the full back-office on web
- **Teachers** — attendance, marks, homework, and their own leave, primarily on mobile
- **Students and parents** — a portal with attendance, results, fees, homework, and messaging
- **Operators** — a super-admin console for managing multiple school tenants

Staff whose own children study at the school can switch between their staff account and their child's portal on the same device, with a password gate when moving back up to staff.

---

## Architecture

### Multi-tenancy
A master database holds the school registry; **each school gets its own dedicated database**. Tenant resolution happens per request from the authenticated user's school code — there is no shared-table filtering, so one school's query cannot reach another's data even if application logic fails.

### White-label mobile apps
Each school gets its own branded Android app — own name, icon, splash screen, colors, package ID, and Play Store listing — generated from a single codebase via per-tenant configuration. A school's API URL is resolved at launch from a central bootstrap endpoint, so a domain change never requires an app rebuild.

### Access control
Authentication is JWT-based. Permissions are **not** embedded in the token; they're fetched separately, so a permission change takes effect without forcing re-login. Every API route independently verifies permission — the UI hides what you can't do, but the server is what enforces it.

Sessions carry a token-version claim, so a password change or forced logout invalidates tokens on every other device.

### Offline support
Attendance marking and marks entry queue locally on mobile and sync when connectivity returns, with idempotency keys so a replayed request never double-writes. This matters in practice: classrooms often have no signal.

---

## Tech stack

| Layer | Technology |
|---|---|
| Web | Next.js 16, React 19, TypeScript, Tailwind CSS |
| Mobile | Expo SDK 55, React Native 0.83, Expo Router, TypeScript |
| Database | PostgreSQL (multi-tenant), Prisma ORM |
| Auth | JWT with token versioning, bcrypt |
| Notifications | Firebase Cloud Messaging |
| Documents | Puppeteer (PDF), SheetJS (Excel) |
| Deployment | Docker, Nginx Proxy Manager, VPS |

---

## Scale

| | |
|---|---|
| API endpoints | 247 |
| Web pages | 80 |
| Mobile screens | 81 |
| Database models | 85 |
| Permission-gated features | 68 |
| TypeScript source files | 667 |
| Documentation files | 179 |
| Automated tests | 400+ across web and mobile |
| Commits | ~400 |

---

## Engineering approach

A few practices that shaped the codebase, included here because they explain the structure more than a feature list does.

**Security auditing as a build phase.** The project went through a systematic security and correctness audit — threat model, per-module review, and a tracked remediation pass. Findings covered multi-tenant isolation, authentication and session handling, input validation, file upload safety, rate limiting, audit-log integrity, and SQL/formula injection in exports. Fixes carry references to the audit item that motivated them.

**Server-side enforcement, always.** Client-side gating is treated as user experience, never as security. Where a feature hides something from a user, the corresponding API returns nothing — so an outdated app build or a direct API call is equally covered.

**Fail-open where hiding data would be worse than showing it.** Configuration flags that control visibility default to the pre-existing behaviour, so deploying a new toggle can never silently blank a working module for schools that never asked for it.

**Regression tests pinned at the level a bug was observed.** When a defect is fixed, the test asserts the user-visible outcome ("can this staff member log in with the password the admin was shown?") rather than the internal call, so the same class of bug can't reappear through a different path.

**Documentation as a first-class artifact.** 179 documents covering architecture, per-module behaviour, API contracts, deployment runbooks, disaster recovery, migration procedures, and the security audit. Non-obvious invariants are documented where they're enforced, not only in prose.

---

## Roadmap

- Public open-source release under a permissive license
- Installation and onboarding guides for schools without dedicated IT staff
- iOS builds alongside the existing Android apps
- Contributor documentation and a plugin approach for school-specific customization
- Internationalization and multi-currency support

---

## Project status and license

Currently in production with live schools while the codebase is prepared for public release — removing deployment-specific configuration, expanding contributor documentation, and finalizing license selection.

The intent is a permissive open-source license, so any school can deploy, modify, and rebrand the platform without cost or restriction.

---

## Contact

This project is being prepared for open-source release. For questions about the platform, deployment, or contributing, please open an issue once the repository is public.
