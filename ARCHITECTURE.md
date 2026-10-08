# Architecture

A technical overview of how the platform is put together, and why. No code — this is the map, not the territory.

---

## 1. Multi-tenancy: database-per-school

Most SaaS platforms achieve multi-tenancy with a `tenant_id` column and a `WHERE` clause on every query. One forgotten filter leaks another customer's data. For school records — attendance, fees, exam results, and children's personal details — that risk was not acceptable.

This platform instead gives **each school its own PostgreSQL database**.

```
                    ┌─────────────────┐
                    │  Master DB      │
                    │  ─────────────  │
                    │  school registry│
                    │  super admins   │
                    │  connection info│
                    └────────┬────────┘
                             │  resolves by school code
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
   ┌────────────┐     ┌────────────┐     ┌────────────┐
   │ School A DB│     │ School B DB│     │ School C DB│
   │ students   │     │ students   │     │ students   │
   │ fees       │     │ fees       │     │ fees       │
   │ exams …    │     │ exams …    │     │ exams …    │
   └────────────┘     └────────────┘     └────────────┘
```

The master database stores only the school registry and its connection details. Every request resolves its tenant from the authenticated user's school code and obtains a client scoped to that school's database.

**The consequence that matters:** a query bug cannot cross a tenant boundary, because the connection itself has no access to the other database. Isolation is enforced by the infrastructure rather than by remembering to write a filter.

The trade-off is operational: schema changes must be applied to every tenant database. This is handled through a controlled per-tenant push flow, and it's a deliberate exchange of deployment convenience for isolation guarantees.

---

## 2. Authentication and authorization

### Identity

Three principal types authenticate against a tenant:

- **Students/parents** — by admission number, or by a registered mobile number
- **Staff** — by username, or phone number for staff without a full user account
- **Super admins** — against the master database, for provisioning and operations

Sessions use JWTs with a fixed lifetime and a `tokenVersion` claim. Changing a password or forcing a logout increments that version, which invalidates every token issued earlier — so a password change actually ends other devices' sessions rather than merely changing a future login.

The token also records **which table authenticated it**. Staff can exist as either a user record or an employee record, and those tables have independent ID sequences; without recording the source, a numeric ID collision can validate a session against the wrong row. That claim exists because the collision happened in production.

### Permissions

Permission keys follow `module.feature.action` — for example `fees.dues.view` or `exams.marks.edit`. There are **73 permission-gated features with 231 permission keys**. The catalogue is one list on the server, served to both apps, and a test fails the build if code uses a key the catalogue lacks, or the catalogue carries a key nothing uses. When a key is split into a finer one (for example, separating "export passwords" from "view students"), every role that held the old power is granted the new key automatically, so no one silently loses access.

Permissions are deliberately **not** embedded in the JWT. They're fetched from a dedicated endpoint after login, which means:

- an administrator can change a role and it takes effect without forcing users to log out
- tokens stay small enough to fit comfortably in proxy header limits

Every API route checks its own permission. The UI hides what a user can't do, but that's presentation — the server is the enforcement point. A user with a stale app build or a crafted request gets the same answer as one using the UI correctly.

### Class-teacher scoping

Some capabilities aren't role-based but relationship-based. A class teacher may mark attendance and approve leave — but only for their own class **and section**. Scoping keys on the `(class, section)` pair, so 10-A's teacher cannot act on 10-B, and a request that omits the section is refused rather than defaulted.

---

## 3. Applications

### Web (Next.js 16 / React 19)

The administrative surface: 93 pages covering every module, plus a student/parent portal and a super-admin console. Server-side API routes handle all data access; the browser never talks to the database.

### Mobile (Expo SDK 55 / React Native)

99 screens covering the staff workflows that happen away from a desk — attendance, marks entry, homework, tasks, leave — and the full student/parent portal.

Each school receives its **own branded app**: name, icon, splash screen, theme color, Android package ID, and Play Store listing, all generated from one codebase through per-tenant configuration files. Adding a school is a configuration change and a build, not a fork.

At launch, the app resolves its school's API URL from a central bootstrap endpoint. A school changing domains needs no rebuild and no store resubmission — the operator updates one field.

---

## 4. Offline capability

Classrooms frequently have no usable signal, and attendance is exactly the task performed there. Attendance, fee collection, leave requests and tasks work offline:

1. The action is written to a local queue and the UI confirms immediately
2. When connectivity returns, the queue replays against the server
3. Each queued request carries an **idempotency key**, so a retry or duplicate flush creates one record, not several

The user-visible promise is that data entered without signal is never silently lost. A queue viewer in the app shows anything still pending.

---

## 5. Notifications

Push delivery uses Firebase Cloud Messaging, with device tokens bound to identities rather than to a single account. This handles the real-world family device:

- one phone can be registered for **several siblings**, so any child's notification reaches the parent
- a staff member whose children study at the school receives both their staff notifications and their children's
- the send pipeline **deduplicates by device token**, so a shared phone alerts once per event rather than once per matching identity

Browsers and iPhone home-screen apps receive the same notifications through web push, with guards so a shared computer never shows one school's or one person's notification to another. Each school can switch notifications off entirely or one kind at a time (fee receipts, attendance, messages and so on), enforced at the single point every notification passes through.

Notifications carry the identity they belong to. Tapping one switches to the correct profile before navigating — passwordless when descending to a child, password-gated when returning to a staff account.

---

## 6. Money handling

Financial correctness carried the strictest requirements in the project.

**Atomic collection.** A fee payment updates dues, creates a receipt, posts to accounting, and — for UPI — flips the payment request to approved, all inside a single database transaction. Partial failure is not possible.

**Race safety.** Operations that move money use compare-and-set guards on the state they read. Two administrators clearing the same cheque simultaneously: one succeeds, the other is rejected and told to reload. The alternative — both passing an in-memory status check — double-posts the income.

**Reversibility with integrity.** Cheques can be cleared, declined, or a mistaken clearance undone. Reversal restores the prior state rather than deleting records, so receipts and audit history survive. Deleting would orphan the income and fee records the cheque paid for.

**No negative outcomes.** Fare revisions and waivers floor at the amount already paid, so reducing a fee never produces a negative balance or an implied refund.

---

## 7. Audit logging

Two layers:

- **An automatic change log.** Every create, update and delete on every school table is captured at the database-client level — a full before-and-after copy of each record, the fields that changed, who did it, from which device and screen. No feature can forget to log, because no feature writes the log; a bulk change of 2,000 rows logs 2,000 rows. Entries are kept only if the change actually commits, are written after the reply so logging never slows or fails a save, and secrets are masked. Viewing is never logged.
- **A business log** for context the data alone doesn't carry ("fee collection, receipt R/…"), joined to the change log by request.

Business-log records are chained with a **tamper-evident hash** — each entry incorporates the previous entry's hash, so removing or editing a historical row invalidates everything after it. For fee collection and grade changes, that detectability matters more than the log itself.

---

## 8. Documents and exports

Schools run on paper output, so exports are treated as a first-class feature rather than an afterthought.

- **Excel** — styled workbooks matching the layouts schools already use, including register-style fee statements with class bands, subtotals, and due-date column groups
- **Printing and PDF** — report cards, receipts, bills, transfer certificates and gate passes are rendered by the user's own browser, or made into a PDF on the phone itself. There is no server-side PDF service; fonts are embedded so a card lays out identically on every device
- **Templates** — transfer certificates, report cards and tabulation sheets are HTML templates the school can import and edit without code changes or an app release

Exports sanitize cell content against formula injection, since spreadsheets execute what looks like a formula and these files are opened on school office machines.

---

## 9. Testing and verification

About 1,750 automated tests across web and mobile, weighted toward the areas where a silent failure is expensive: fee arithmetic, discount stacking, permission resolution, multi-tenant isolation, authentication and session invalidation, timezone handling, and export integrity.

The guiding practice: **a regression test asserts the user-visible outcome, not the internal call.** When a bug is fixed, the test states the property that was violated — "can this staff member log in with the password the admin was shown?" — so the same class of failure can't return through a different code path. Several tests exist specifically because they failed first and proved a fix wrong before it shipped. Every printed total is tested against the rows printed beneath it — a class of bug that was found four separate times before that rule existed.

---

## 10. Security posture

The project underwent a structured security audit — threat model, per-module review, and a tracked remediation pass. Areas covered:

| Area | Concern |
|---|---|
| Tenant isolation | Cross-school data access |
| Authentication | Session lifetime, revocation, identity collisions |
| Authorization | Server-side enforcement on every route |
| Input validation | Typed schema validation at API boundaries |
| File uploads | Type, size, and magic-byte verification |
| Rate limiting | Login, payment initiation, password change |
| Audit integrity | Hash-chained, tamper-evident |
| Injection | SQL via ORM; formula injection in exports |
| Secrets | Encrypted at rest, never logged |

Two principles run through the fixes:

**Server-side enforcement, always.** Where the UI hides something, the API returns nothing. An old app build gets the same protection as a current one.

**Fail open on visibility, closed on authority.** A configuration flag controlling what a user *sees* defaults to the pre-existing behaviour, so deploying a new toggle can't silently blank a working module. A check controlling what a user *may do* defaults to denial.

---

## 11. Operations

- **Deployment** — Docker containers behind Nginx Proxy Manager on a VPS
- **App updates** — the Android apps check Google Play for a newer version, and each school can set a minimum version that forces an update
- **Backups** — scheduled per-tenant database backups with off-site replication and generational retention
- **Disaster recovery** — documented restore procedures with a rehearsal schedule
- **Migrations** — controlled per-tenant schema push with documented runbooks for changes needing data migration
- **Monitoring** — structured error logging with request context

Operational documentation covers the paths that are hard to reason about under pressure: restoring a single tenant, rotating encryption keys, and recovering from a partially applied migration.
