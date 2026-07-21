# Roadmap

What's done, what's next, and what stands between the current state and a platform any school can adopt on its own.

---

## Where the project is now

**In production.** Live schools, real students and staff, daily attendance, fee collection, examinations, and parent communication. Branded Android apps published on the Play Store.

The system works. The gap isn't functionality — it's everything that lets a school adopt it **without the person who built it.**

---

## Phase 1 — Preparing for public source release

*In progress.*

**Separating deployment configuration from application code.**
Credentials, tenant definitions, signing keys, and host-specific settings currently live alongside the code. All of it needs to move behind configuration with documented examples, and the git history needs to be rewritten so nothing sensitive survives in past commits. This is the single blocking item.

**Installation path for non-experts.**
A school administrator is not a DevOps engineer. The target is: bare VPS to running, branded instance by following a document — with a scripted setup for the common case and a manual path for the rest.

**Contributor documentation.**
Architecture decision records explaining *why* the multi-tenant model, the offline queue, and the permission design are shaped the way they are. Module guides. A codebase navigable by someone who didn't write it.

**License and governance.**
Apache-2.0 is chosen. Still needed: contribution guidelines, a code of conduct, security disclosure policy, and a clear statement of what the project will and won't accept.

---

## Phase 2 — Making adoption realistic

**Onboarding a school without the author.**
The real test of an open-source project is whether a stranger can deploy it. This means seed data, a guided first-run that creates the academic session and classes, sample fee structures, and an import path from the spreadsheets a school already keeps.

**Demo instance.**
A public sandbox with fictional data, so a school can evaluate the platform before committing to a server.

**iOS builds.**
The mobile apps are Android-only today. The codebase is cross-platform; the work is Apple provisioning, per-tenant build configuration, and App Store review — meaningful effort, no architectural change.

**Internationalization.**
Currently India-specific by design: April–March academic sessions, Indian numbering in currency display, UPI payments, and CBSE-shaped examination structures. Making session models, currency, date formats, and payment integrations configurable opens the platform to schools elsewhere.

---

## Phase 3 — Sustainability

**Plugin architecture for school-specific customization.**
Every school differs in fee structure, grading scheme, and report format. Today those differences would require a fork. A plugin approach — custom report templates, pluggable fee rules, configurable grade calculations — lets a school adapt the system without diverging from upstream.

**Continued security review.**
The threat model was built for a private deployment. Public source widens it: attackers can read the code, and self-hosting schools will vary enormously in operational competence. Secure defaults, hardening guides, and an ongoing review cadence matter more once the code is public than they did before.

**Test coverage expansion.**
The suite is strongest where mistakes cost money and weakest at the edges. Fee arithmetic, permission resolution, and multi-tenant isolation deserve deeper property-based coverage — these are the places where a silent error surfaces as a wrong number on a receipt rather than a crash.

**Performance at scale.**
Current tenants are hundreds of students. The architecture should hold at thousands, but "should" is not "verified." Load testing, query analysis, and index review before a large school discovers a limit in production.

---

## Deliberately not planned

Stating these keeps the scope honest:

- **Hosted SaaS offering.** The point is that schools self-host and own their data. A paid hosted version would recreate the problem the project exists to solve.
- **Per-student licensing, in any form.** Free means free.
- **Feature parity with every commercial ERP.** Library management, transport GPS tracking, and biometric integration are all reasonable asks and all better served as plugins than as core.

---

## Where Claude Max fits

The remaining work is not conceptually hard. It's a large volume of careful, unglamorous engineering — and it's precisely the work that decides whether this becomes a platform other schools use or stays a system that runs well for three of them.

**Documentation for non-technical adopters** is the largest single gap. Installation guides, troubleshooting, and operational runbooks written for a school administrator rather than a developer.

**Contributor onboarding** — architecture decision records, module guides, and the codebase readability work that lets someone else contribute meaningfully.

**Security hardening** as the threat model widens with public source.

**iOS builds and internationalization** to reach beyond the current Android, India-focused deployment.

**Test coverage** in the areas where silent errors are expensive.

The bottleneck on this project has never been ideas. It's the distance between working software and adoptable software — and that distance is measured in documentation, hardening, and careful review rather than new features.
