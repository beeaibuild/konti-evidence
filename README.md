# Konti — AI-assisted operations system for container rental and transport

> **About this repository.** This is a public companion to Konti, whose source code is in a private repository.
> It contains an English translation of the project README, a curated English evidence summary
> ([EVIDENCE.md](EVIDENCE.md)) and copies of the referenced technical documents in their original Hungarian (`docs/`).
> Links to `github.com/beeaibuild/kontener-app` (PRs, CI runs) point to the private repository and do not open publicly.
> In the public copies, the production domain, the operator's name and development seed credentials have been removed.

Konti is a web-based mini ERP for the daily operation of a container service and a small transport fleet.
It connects orders, dispatch planning, driver tasks, communication, digital documents
and AI recommendations that are approved by a human.

## Current state — 20 November 2026

The verified `main` is `53fc1ff` (#213); the most recent application code change is `46c470b` (#212).
The 20 November update is a GitHub source and PR check; no new Render check or physical tablet test took place.
"In main" below means merged code; deployment and physical tablet testing are confirmed by separate evidence.
Detailed history: [development log](docs/DEVELOPMENT-LOG.md) (Hungarian). English summary of the evaluation work: [EVIDENCE.md](EVIDENCE.md).

| Area | Current, traceable state |
| --- | --- |
| Manual order entry from text and voice | #188 in main. Editable transcript, personal Konti context, field-level source and feedback. Green: stated data; blue: completed from context; red, empty: required data missing. Saved only after the operator reviews it. |
| Order list | #194–195 in main: full delivery address, today's Budapest date as default, descending order number. |
| Partners and company orders | #196–199 in main: separate ordering parties even with a shared email, manual partner change, private person/company selector, company name, tax number and contact person. |
| Placement correction | #200–201 in main: change of address, container size and assigned container; server-side price/route preview, re-check on save, transactional booking swap, task update, audit and notification event. |
| Shared communication | #199 in main: an "All drivers" conversation independent of days. |
| Idle alert | #202 in main: during an active GPS shift between 07:00 and 17:00 Budapest time, a 30-minute stop confirmed by continuous measurements within a fixed 200 m circle; a one-time sound/admin message and GPS marker. Works while the admin interface is open. |
| Konti Driver APK | #193: signed 2.0.2 (5), update link that can be shared in chat. Production deploy and download/hash check confirmed on 15 November 2026; physical Samsung update still open. |
| AI telemetry | #190 and #192 in main: weekly frozen summary, append-only storage, SHA-256, JSON/Markdown export; schema 2 also counts VERIFIED initial assignment decisions. |
| 17:00 tablet sign-out | #203 in main and in the current production release: shift closes at 17:00 Budapest time; for a normal/internal trip active at that moment, closing is deferred until a GPS stop of 30 minutes within 200 metres. Dispatcher and browser admin sessions remain separate. Verification of the first evening run in production is still open. |
| Registered partners | #211 in main and in production: registrations without orders and order activity; direct account link, otherwise an unambiguous email match at order time. |
| AI statistics display | #212 in main and in production: separate counts for same-day and next-day recommendations. Acceptance and execution metrics refer to next-day recommendations; same-day details are shown separately. |

### Evidence and open checks

- #188: CI/P63 on the tested source tree: 3586 Vitest and 177 UI tests, production and Docker build.
- #193: 2330 local unit tests, typecheck, production build, Android unit/release build,
  APK signing and HTTP/hash check. Production LIVE: 15 November 2026, 15:13 (Budapest).
- The test counts above belong to the given release; they are not results of re-running today's full `main`.
- Open: on a Samsung tablet, the 2.0.1 → 2.0.2 update, pairing, login, tasks,
  background GPS and push. Silent installation and Play Protect exemption are not confirmed.
- Last documented Render production check (17 November 2026): `46c470b277cdf3c4f95f6722e88e80cafb650067`, deploy
  `dep-dbu3fvh42hec73bfj280`, LIVE 17 November 2026, 12:21:50 (Budapest).
  The deploy record confirms the deployment; no new functional or physical tablet test took place now.
- #209 still open: server-side acknowledgement of the old APK's GPS queue that builds up
  after automatic shift closing. The production commit above does not contain this fix.
- #214 still open, built on the #209 branch: authority-checked acknowledgement, without storage,
  of points arriving after GPS shifts closed on the server side because of a hard expiry.
  New `GpsShift.serverClosed` field and Prisma migration; existing rows default to false.
  According to the PR description, migration, typecheck and 37/37 integration tests passed
  on PostgreSQL 18.6; this was not re-run now. Not yet merged into main, no physical tablet test.
  After #209 is settled, it needs to target main and pass the required gates for the current main.
- #208 still open: post-hydration check in the route preview Playwright test.
- #205 and #207 in main: required CI gates, narrowed checks for documentation PRs,
  and an exact main SHA and check verification before the manual P63 deployment step.
  Procedure: [main merge and deployment](docs/main-merge.md).

This 20 November documentation check produced no new evidence for the physical tablet
and evening/morning operational checks previously marked as open.

### Tablet login and daily availability (16 November 2026)

A successful new tablet login adds the driver and the tablet's truck to the schedule for the
Budapest calendar day. It keeps resources already assigned; if there is no plan for the day, it creates one.
Signing out and shift closing do not remove them from the daily schedule. Availability for the next
day comes from a new daily login or manual assignment; planning ahead remains possible.

A failed login does not change the schedule. Resending an identical login request does not overwrite
a manual change made in the meantime. No new migration or dependency, and no APK update is needed.
Login, schedule and daily chat permission updates happen in the same transaction.
Once the first plan of the day is created, the usual daily assignment limit applies:
drivers and trucks in that plan can be selected.

Release check: 2347/2347 unit tests, 77/77 PostgreSQL integration tests,
typecheck, production build and line-length gate PASS. The PostgreSQL run used an isolated 18.6
test database with all migrations. The production service uses PostgreSQL 16.
Production LIVE: 16 November 2026, 18:14:44 (Budapest), `c37e2ea`.
At 18:15, health/database/commit checks on the production domain and on the direct Render host PASS.
Physical tablet login still to be verified.

### AI and measurement limits

The model recommends, the server checks permissions and business rules,
and the operator decides on execution. The developer tests and historical benchmarks
of Viber message interpretation are not general accuracy measurements. The latest documented
18-case held-out set gave automatic results of 6/18, 5/18 and 5/18 over three runs;
4 cases stable, 2 fluctuating, 12 failed. The cases seen have become development data.

Details: [order capture](docs/order-capture-v1.md),
[Viber address context](docs/viber-order-address-context-v1.md),
[measurement protocol](docs/viber-eval-run-protocol.md),
[two-task exchange](docs/two-task-exchange.md),
[cascade analysis](docs/cascade-impact.md), [cascade monitor](docs/cascade-monitor.md).
The current switch state of the cascade and proactive features must be checked in the target environment.

## Features

| Area | Capabilities implemented in the repository |
| --- | --- |
| **Customer and order** | Public container ordering, price estimate, registration and customer account, tracking own orders, cancellation and lifecycle notifications, "container full" signal. |
| **Admin and dispatcher** | Order and task management, manual orders, containers, drivers and vehicles, partners, internal transport tasks, daily plans and sequence, weekly capacity, dispatch map. |
| **Driver** | Own daily jobs and status reports, mobile and landscape tablet view, shared operational chat, opening and closing the digital delivery note. |
| **Communication** | Role-protected shared daily chat, replies and private image attachments, administrable write permissions, notification/email flows. |
| **AI decision support** | Limited, read-only AI context and chat; recommendations for reassigning overdue tasks and for assigning **same-day** planned tasks; confirmed execution; decision feedback and 7/30/90-day measurement. |
| **GPS and route** | Native Android GPS with offline buffer and device pairing; the unified driver app also opens the web driver interface. The daily recommendation can take into account the work sequence of the same vehicle, sufficiently fresh GPS data and a configured road route estimate. |
| **Documents and finance** | PDF delivery note closed with customer signature, audited email recovery; payment statuses and mock payment; invoicing data preparation and admin interface. |

**Important limits:** the payment API currently contains a mock path; the existence of the invoicing panel does not prove automatic, live invoice issuing via Számlázz.hu. The GPS companion and route-based AI depend on pilot/environment settings: green CI alone does not prove physical Android reliability or real logistics savings. The Google route estimate in same-day AI recommendations is a snapshot that takes current traffic into account, with departure at request time on every leg. It is not a planned arrival time or a guaranteed travel time; loading/unloading time and traffic at later departures are not modelled. The distance call used for pricing remains traffic-free.

## Operating principles

1. **Human at the decision point:** the model recommends only from properly generated candidates; acceptance, preview, a separate execution confirmation and a logged result are required. Wrong or stale context cannot turn into a silent assignment.
2. **Business integrity:** order, task, container and dispatcher operations run with server-side validation, transactions, audits and a defined lock order.
3. **Permissions and data minimisation:** active OPERATOR / DRIVER / CUSTOMER / MASTER_ADMIN modes, revocable sessions, own driver and customer scope; the AI does not automatically receive raw personal or GPS coordinate data.
4. **Measured, not assumed, state:** we separate features that are in the code, confirmed by tests, tried on staging and proven in production.

Details: [operational recommendations](docs/operational-recommendations.md), [same-day GPS/route-based recommendation](docs/same-day-assignment-recommendation.md), [transaction lock protocol](docs/transaction-lock-protocol.md), [GPS pilot](docs/android-gps-pilot.md).

## Architecture

**Core stack:** Next.js 15.5.20 (App Router), React 18, TypeScript, Prisma 5.22, PostgreSQL 16, Zod, Tailwind CSS. Testing uses Vitest + Playwright; deployment is Docker-based, with Render staging/production services.

**Additional components:** configurable Google Routes/Maps integration; server-side email gateway; R2 for private image attachments; PDF generation with `pdf-lib`; `mobile/gps-companion` Capacitor Android app with native background location and a local SQLite buffer; the unified driver app also opens the web work interface inside it.

| Directory | Role |
| --- | --- |
| `src/app/` | Next.js pages and API routes |
| `src/services/` | Business services, AI, GPS, recommendations, documents |
| `src/lib/` | Shared permission, validation, date and transaction helpers |
| `prisma/` | Schema and versioned database migrations |
| `mobile/gps-companion/` | Android GPS and unified Driver V1 app |
| `tests/`, `e2e/` | Unit, integration and end-to-end tests |
| `docs/` | Technical decisions, pilots, release and operations documentation |

## Operations and current evidence

- Staging and production are **separate environments**. Before a release, a backup of the target system, a migration check and a smoke test of the service's matching commit are required.
- The source gate of the [#110 release](https://github.com/beeaibuild/kontener-app/pull/110) records **3034 Vitest, 173 UI and 5 delivery note E2E** results; this is a **historical** number for that specific release, not a permanent test count.
- For health monitoring the application includes an `/api/health` endpoint, machine-readable server logs and a business audit.
- **CI success must not be confused with production deploy success.** The basic GitHub Actions CI checks unit tests, type checking and the production build; native GPS, delivery note and other targeted tests appear in separate gates.

Related documents: [production bootstrap](docs/production-bootstrap-p39.md), [backup/restore proof](docs/backup-restore-proof-p36.md), [digital delivery note](docs/digital-delivery-note-v1.md), [GPS pilot](docs/android-gps-pilot.md).

## Documentation and history

- [**Evaluation and verification evidence**](EVIDENCE.md) — curated English summary.
- [**Development log**](docs/DEVELOPMENT-LOG.md) — milestones, decisions, evidence, open points (Hungarian).
- [AI recommendations and execution](docs/operational-recommendations.md) · [AI generation](docs/operational-recommendation-generation.md) · [feedback/telemetry](docs/operational-recommendation-metrics.md).
- [Operations / migration](docs/migration-workflow.md) · [security logging](docs/logging-health-p33.md) · [tablet and GPS](docs/android-gps-pilot.md).

**Among the documents, this README is the product's entry point and the development log is the history of changes; the PR and the target environment's test/deploy evidence are the source of the current state.**
