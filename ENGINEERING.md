# Engineering Practices

How this project is built and verified. Included because a feature list doesn't tell you whether software can be trusted with a child's grades or a family's fee payments.

---

## The systematic audit

Before the platform went to production with real schools, it went through a structured audit across **40 categories**, each reviewed independently with findings tracked to resolution.

| | | |
|---|---|---|
| Authentication & session | Multi-tenancy isolation | Permission enforcement |
| Input validation | Output encoding | SQL injection |
| SSRF & path traversal | Cryptography | HTTP headers |
| Rate limiting | Mobile security | UPI payment flow |
| Money math | Concurrency | Audit trail |
| Soft delete | Error handling | Web performance |
| Mobile performance | PDF reliability | Notification reliability |
| Mobile offline | Web quality | Mobile quality |
| iOS HIG compliance | Android MD3 | Web accessibility |
| Mobile accessibility | Design system | Cross-platform parity |
| Database schema | Backup & restore | VPS hardening |
| Proxy configuration | Dependency CVEs | Test coverage |
| Test quality | Documentation drift | Observability |
| API version skew | | |

**326 findings** were catalogued and prioritized P0–P3. **233 of them are still cited by ID in code comments** at the exact line where the fix lives — so a future maintainer reading a defensive-looking check finds out *which* attack or failure motivated it, rather than deleting it as unnecessary.

A representative example, paraphrased from a real comment:

> Pre-fix, this issued two independent writes with no transaction and no compare-and-set guard. Two concurrent waives both read `waivedAmount=0`, computed the same new value, and the second write overwrote the first — one waive lost on the fee, but *both* discount rows persisted, so the ledger double-counted. Now the update is guarded on the read snapshot; a concurrent writer causes a rollback and a 409 so the UI reloads.

That comment is worth more than the three lines of code it sits above.

---

## Security model

A full threat model is maintained as a living document — assets ranked by impact-if-compromised, trust boundaries, attacker profiles, and mitigations.

**Assets ranked highest-impact first:** student PII, published marks, fee and payment history, the JWT signing secret, the raw-password encryption key, tenant database credentials, audit-log integrity, push-notification credentials, and backups.

**Principles the code actually follows:**

**Server-side enforcement, always.** Client-side gating is user experience, never security. Where the UI hides a feature, the corresponding API returns nothing — so an outdated mobile build or a hand-crafted request gets exactly the same answer as the UI would give.

**Fail open on visibility, closed on authority.** A flag controlling what a user *sees* defaults to the previous behaviour, so shipping a new toggle can't silently blank a working module for schools that never asked for it. A check controlling what a user *may do* defaults to denial.

**Isolation by infrastructure, not by discipline.** Tenants are separated by database, not by a `WHERE` clause. Forgetting a filter is a bug every codebase eventually has; here it cannot cross a school boundary because the connection has no access to the other database.

**Detectability where prevention isn't enough.** Audit records are hash-chained, so altering history invalidates every subsequent entry. For fee collection and grade changes, being able to prove tampering matters more than hoping to prevent it.

---

## Correctness where mistakes cost money

Fee handling took the strictest treatment in the project.

**Atomic operations.** Collecting a payment updates dues, writes a receipt, posts to accounting, and — for UPI — approves the payment request, all in one database transaction. There is no partial state.

**Race safety by compare-and-set.** Money-moving operations guard their update on the state they read. Two administrators clearing the same cheque at the same moment: one wins, the other is refused and told to reload. Without the guard, both pass an in-memory status check and the income posts twice.

**No negative outcomes.** Fare reductions and waivers floor at the amount already paid, so lowering a fee never produces a negative balance or an accidental refund.

**Reversal over deletion.** A cheque cleared by mistake can be undone — moving the money back and restoring the prior state — but the record is never deleted, because the linked income and fee transaction would be orphaned and the receipt would lose its trail.

---

## How bugs get diagnosed

Several production issues in this project looked like one problem and turned out to be another. The working rule became: **measure before changing code.**

**Case: "some staff can't log in."**
The symptom was a login failure and a password reset that didn't help. It turned out to be *two unrelated bugs with identical symptoms.* One group of staff had a legacy account row that login checked first while the admin reset wrote to a different row — so the password shown in the UI could never work, and resetting again could never fix it. A different staff member had a healthy account but had changed their own password, which left the admin screen displaying a stale credential that had silently stopped working. Same error message, entirely different causes, different fixes.

**Case: "256 conflicts found."**
An alarming red badge on the sibling-management screen. Measuring first showed **418 of 429 flagged pairs were already correctly linked** and working — the report ignored the discriminator that the login path actually uses. The genuine worklist was nine students. More importantly, two of the remaining cases were *not* families at all but unrelated households sharing a mistyped phone number. Bulk-approving the panel — the obvious "fix" — would have given one parent access to another family's child records. The real fix was filtering the report and adding a warning that blocks exactly that mistake.

**Case: a logo missing from an exported PDF.**
Fixed once with a change that did nothing: a helper was handed a path it internally re-prefixed, so the lookup silently failed and left the original broken value in place. Writing a test against a real file proved the "fix" wrong before it reached anyone. The corrected version shipped only because the test failed first.

The pattern in all three: **the obvious fix would have been wrong**, and in one case actively harmful.

---

## Testing

400+ automated tests across web and mobile, weighted toward areas where silent failure is expensive: fee arithmetic, discount stacking, permission resolution, multi-tenant isolation, session invalidation, timezone handling, and export integrity.

**Tests assert the user-visible property, not the internal call.** After the login bug above, the regression test doesn't check that a particular function was invoked — it asks *"can this staff member log in with the password the administrator was shown?"* That phrasing is what makes the test survive refactors and catch the same class of bug arriving through a different path.

Several tests exist specifically because they **failed first and disproved a fix**. That's the point of writing them before declaring victory.

---

## Documentation

179 documents covering architecture, per-module behaviour, API contracts, deployment runbooks, disaster recovery, migration procedures, and the security audit.

Two conventions keep it useful rather than decorative:

**Invariants are documented where they're enforced.** A non-obvious rule gets a one-line comment at the code that depends on it, with the reason — not a paragraph in a wiki nobody opens.

**Runbooks target the situations that are hard to think through under pressure:** restoring a single tenant without touching the others, rotating an encryption key, recovering from a half-applied migration.

---

## Operational maturity

- **Deployment** — containerized, behind a reverse proxy, on a VPS a school can own
- **Backups** — scheduled per-tenant database backups, off-site replication, generational retention
- **Disaster recovery** — documented restore procedures with a rehearsal schedule
- **Migrations** — controlled per-tenant schema push, with runbooks for changes needing data migration
- **Release discipline** — versioned mobile builds per tenant, published to the Play Store

The platform has been through real production incidents — a boot-crashing release caused by an over-eager dependency cleanup, an authentication regression from an ID collision between two tables, a fee module that silently stopped generating charges for a newly created tier. Each was diagnosed, fixed, covered by a test, and documented. That history is why the practices above exist; none of them were adopted in the abstract.
