# Project Overview

Context for reviewers: what this project is, why it exists, what state it's in, and what open-sourcing it is meant to achieve.

---

## The problem

Most K-12 schools outside the well-funded tier run their administration on paper and spreadsheets. Attendance in registers. Fees in ledgers. Marks in Excel files emailed between teachers. Announcements in WhatsApp groups.

The failures are mundane and constant:

- A parent learns their child was absent at the monthly meeting, three weeks late
- A fee receipt exists as one carbon copy; a dispute has no second source
- Report card season costs a week of manual arithmetic and produces errors nobody can trace
- "How many students haven't paid August transport?" takes an afternoon
- A teacher's marks spreadsheet is the only copy, on one laptop

Commercial ERP platforms solve this — for schools that can afford per-student annual licensing, accept that the vendor holds their data, and can wait on a support ticket for a change to a report format. For everyone else the honest options are paper or nothing.

## The approach

A **free, open-source, self-hostable, white-label** ERP that a school can own outright.

- **Self-hostable** — the school's data lives on infrastructure the school controls
- **White-label** — the school's name, logo, colors, and its own app on the Play Store
- **Multi-tenant** — one deployment serves many schools, so a district or operator can host several
- **Free** — no per-student fee, ever

The design premise is that a school should not have to choose between affordability and owning its own records.

---

## Current state

**In production.** Running for real schools — roughly 900 students and 70 staff on the largest tenant — handling daily attendance, fee collection, examinations, and parent communication. Branded Android apps are published on the Play Store for each school.

This is not a prototype or a demo. Features have been shaped by production incidents: the multi-tenant model, the offline queue, and several authentication fixes all exist because of problems encountered with live users and live money.

| | |
|---|---|
| API endpoints | 247 |
| Web pages | 80 |
| Mobile screens | 81 |
| Database models | 85 |
| Permission-gated features | 68 |
| TypeScript source files | 667 |
| Documentation files | 179 |
| Automated tests | 400+ |
| Commits | ~400 |

---

## What makes it different

**Database-per-tenant isolation.** Each school gets its own database, not a shared table with a tenant column. A query bug cannot cross a school boundary because the connection has no access to the other database.

**Offline-first where it matters.** Attendance and marks entry work without signal and sync later, with idempotency so replays don't double-write. Classrooms often have no connectivity, and those are exactly the tasks performed there.

**Real financial rigor.** Fee collection is transactional and race-safe. Reversals restore state rather than deleting records. Fare reductions floor at amounts already paid. Audit logs are hash-chained and tamper-evident.

**White-label mobile apps from one codebase.** Adding a school is configuration plus a build, not a fork.

**Documentation treated as product.** 179 documents covering architecture, module behaviour, deployment, disaster recovery, and the security audit — because a school without IT staff can't read source to find out how backups work.

---

## Development approach

Built with heavy use of Claude Code throughout — architecture decisions, implementation, debugging, security review, and documentation.

Two working practices are worth stating, because they shaped the result more than any tool did:

**Diagnose before fixing.** Several production issues in this project looked like one thing and were another. A "students can't log in" report turned out to be two unrelated bugs with identical symptoms. A "256 conflicts" alarm turned out to be 97% false positives from a reporting query that ignored the discriminator the login path actually uses. The habit of measuring before changing code prevented fixes that would have been wrong and, in one case, would have exposed one family's child records to another.

**Verify the fix, don't assume it.** A logo missing from an exported PDF was "fixed" once with a change that did nothing — a helper was passed a path it double-nested. Writing a test against a real file caught it. The corrected fix shipped because the test failed first.

Both practices are visible in the commit history, which documents root causes rather than just changes.

---

## Why open source

Three reasons.

**The economics are wrong for schools.** Per-student licensing scales cost with exactly the thing schools can't control. A free, self-hostable alternative changes what's possible for a school with no software budget.

**Schools should own their records.** Attendance, grades, fee history, and children's personal data belong to the institution and the families. Self-hosting makes that a structural fact rather than a contractual promise.

**Every school is a little different.** Fee structures, grading schemes, report formats, and session models vary by region and board. Open source lets a school adapt the system instead of waiting on a vendor roadmap.

---

## Roadmap to release

Currently preparing the codebase for public release:

- Separating deployment-specific configuration from application code
- Installation guides aimed at schools without dedicated IT staff
- Contributor documentation and architecture decision records
- License finalization — the intent is permissive
- iOS builds alongside the existing Android apps
- Internationalization and multi-currency support

The goal after release is that a school can go from a bare VPS to a running, branded instance by following documentation — no vendor, no license, no per-student cost.

---

## How Claude Max would be used

Development continues on the areas that stand between the current state and a genuinely usable open-source release:

- **Documentation for non-technical adopters** — the largest gap. A school administrator needs an installation path, not an architecture document.
- **Contributor onboarding** — architecture decision records, module guides, and a codebase readable by someone who didn't write it.
- **Security hardening** — continued review as the code becomes public and the threat model widens.
- **iOS builds and internationalization** — expanding reach beyond the current Android, India-focused deployment.
- **Test coverage** — extending the regression suite, particularly around fee arithmetic and permission resolution, where silent errors are expensive.

The bottleneck on this project has never been ideas; it's the volume of careful work between a system that runs for three schools and one that any school can adopt safely.
