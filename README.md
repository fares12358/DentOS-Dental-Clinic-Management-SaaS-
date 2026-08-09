# DentOS — Dental Clinic Management SaaS

A multi-tenant, production-grade dental clinic management SaaS covering the full clinic lifecycle: patient portal, clinical records, inventory, finance, CRM, real-time chat, platform administration, a public website builder, and custom domain management.

Built solo, end-to-end, across 16 implementation phases and 36 sprints.

---

## Tech Stack

**Frontend:** Next.js, Tailwind CSS
**Backend:** Node.js, Express, MongoDB (Mongoose)
**Auth:** JWT
**Reporting / Exports:** pdfmake
**Dashboards:** Chart.js
**Realtime:** WebSocket-based chat & notification engine
**API:** REST

---

## Architecture

- **Multi-tenant by design:** every module — patient data, inventory, finance, CRM, chat, audit logs, and even clinical tooth-chart data — is scoped per clinic (tenant) and verified isolated both through list-endpoint filtering and direct-ID access attempts.
- **RBAC:** a checkbox-based permission model with feature-flag gating across all 21 backend route modules. A role with zero permissions is blocked everywhere; a role granted exactly one permission unlocks only that resource.
- **Feature flags & plan limits:** toggle live, with no restart required — confirmed to flip a module from `403 FEATURE_DISABLED` to accessible the instant a flag is granted.

---

## Core Modules

### Patient Portal & Clinical Module
Self-service patient signup, verification, login, booking, and doctor chat, backed by a clinical module for session records and tooth-chart data — all clinic-scoped.

### Inventory Management
Full catalog, warehouse, and stock tracking, plus treatment-time consumption, so stock levels reflect what's actually used during patient care.

### Finance
Invoicing, payments, expenses, and cost-of-goods-sold (COGS) tracking tied directly into clinic operations and reporting.

### CRM & Loyalty
Follow-ups and contact history for lead and patient relationship management, plus a loyalty program layered on top of booking and invoicing.

### Real-Time Chat & Notifications
A real-time chat foundation with extensions for staff-to-patient and internal communication, a notification engine, and an AI assistant integration point — fully wired, activates the moment a provider API key is set.

### Platform Administration
Plan-based feature flags and usage limits, monitoring, support tooling, and a full audit log.

### Public Website Builder & Custom Domains
A per-clinic public website builder, with custom-domain management complete and tested on the application side.

---

## Production Readiness

This system shipped with a real, code-grounded production-readiness audit — not a checklist, but executed verification:

- **Backend test suite:** 336 / 336 passing (24 test files, vitest)
- **End-to-end suite:** 21 / 21 passing (Playwright, 4 canonical user journeys — clinic onboarding through booking/clinical session/invoice/payment; a doctor's day with restricted-role navigation; the full patient self-service journey; and platform administration)
- **Backup & restore drill:** executed against seeded data across 10 collections (core clinic data, clinical records, finance, audit logs) — a full backup, restore into a separate empty instance, and verification that both document counts and specific nested field values survived the round trip. **Passed.**
- **Isolation & permission sweep:** a consolidated test suite confirming every feature-flagged module defaults OFF for a fresh clinic and flips live when granted; two-clinic data isolation confirmed across every module; role-based access confirmed at both the zero-permission and single-permission boundary.
- **Two real bugs found and fixed** during end-to-end test authoring — issues that were only reachable through the real UI and validation stack, not the pre-existing API-level test suite:
  - A patient signup schema rejected an empty `phone` string despite the public form explicitly labeling the field optional.
  - A platform config field (`whatsapp.apiKey`) was being stored and re-served in plaintext — closed by encrypting it the same way the rest of the system's credentials are handled (AES-256-GCM).

---

## Project Structure

```
dentos/
├── backend/
│   ├── src/
│   │   ├── modules/                         # 21 route modules, tenant-scoped
│   │   │   ├── auth/
│   │   │   ├── clinics/                     # tenant/clinic core
│   │   │   ├── patients/
│   │   │   ├── clinical/                    # sessions, tooth-chart data
│   │   │   ├── bookings/
│   │   │   ├── inventory/                   # catalog, warehouse, stock
│   │   │   ├── finance/                     # invoices, payments, expenses, COGS
│   │   │   ├── crm/                         # follow-ups, contact history
│   │   │   ├── loyalty/
│   │   │   ├── chat/                        # realtime + AI assistant stub
│   │   │   ├── notifications/
│   │   │   ├── platform/                    # flags, limits, monitoring, support
│   │   │   │   └── platform.config.js       # whatsapp.apiKey (AES-256-GCM)
│   │   │   ├── audit-log/
│   │   │   ├── website-builder/
│   │   │   └── custom-domain/
│   │   ├── middlewares/
│   │   │   ├── validate.js                  # patientSignupSchema, RBAC checks
│   │   │   └── rateLimiter.js
│   │   ├── scripts/
│   │   │   └── backupRestoreDrill.js        # real backup/restore verification
│   │   ├── test/
│   │   │   ├── e2e/
│   │   │   │   ├── bootstrap.js             # mongodb-memory-server setup
│   │   │   │   └── journeys/                # 4 canonical Playwright specs
│   │   │   └── integration/
│   │   │       └── sprint036IsolationMatrix.test.js
│   │   ├── storage/
│   │   │   └── clinical/                    # PHI attachments (isolated from public uploads)
│   │   └── server.js
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── (dashboard)/                 # staff/admin surfaces
│   │   │   │   └── website/
│   │   │   │       └── page.js              # public website builder editor
│   │   │   └── c/
│   │   │       └── [domain]/                # per-clinic public/patient routes
│   │   │           └── (patient)/
│   │   │               └── signup/
│   │   │                   └── page.js      # public patient signup form
│   │   └── middleware.js                    # tenant/domain resolution
│   └── package.json
│
└── docs/
    ├── ai/
    │   └── IMPLEMENTATION_ROADMAP.md        # 16-phase / 36-sprint plan
    ├── ops/
    │   ├── deployment-runbook.md
    │   └── custom-domain-runbook.md
    └── reports/
        ├── system.txt                       # technical reference, Parts 1-4
        ├── sprint036-audit-findings.md
        └── dentos-audit-2026-08-05.txt      # this production-readiness audit
```

> Structure reconstructed from the module names, test files, and paths referenced throughout the production-readiness audit — not a literal `tree` export, so some folder names are inferred from their described role rather than confirmed against the live repo.

## Engineering Practices

- Consistent authorization coverage across all 21 backend route files — no gaps found in audit.
- Every public POST endpoint covered by rate limiting.
- PHI (patient health data) storage confirmed isolated from the public uploads mount.
- Centralized error handling distinguishes expected business errors from unexpected crashes and never leaks internals in production.
- No secret values are ever written to a log line anywhere in the codebase.
- Self-documenting code — inline comments record prior architectural decisions and known deferrals, kept current across all 36 sprints.

---

## Author

Built and maintained solo by **Fares Mohamed** — architecture, all 21 backend route modules, the multi-tenant RBAC/feature-flag system, the full clinical/finance/CRM/chat feature set, and the production-readiness pass itself (test suite authorship, backup/restore drill, documentation reconciliation).
