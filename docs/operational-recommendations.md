# Durable overdue reassignment recommendations — V1

Internal `storeOperationalRecommendation(structuredInput, signedSessionToken)`
accepts one typed `OVERDUE_REASSIGNMENT_V1` action. It never parses chat/model
text. Source MODEL/RULE and isTest are explicit; rule/prompt/model/feedback
versions may be absent. No model, ranking, UI or feedback subsystem is added.

## Persistence and lifecycle

OperationalRecommendation owns immutable action, decision day, explicit target
day, relevant snapshot and SHA-256 hashes. generationRequestId is unique;
canonical identical input returns the same row, differing input conflicts. All
object keys are sorted; arrays preserve business order. Stored plans, snapshots
and generation identity are protected by a DB trigger and never silently revised.

PENDING → ACCEPTED / REJECTED / STALE; ACCEPTED → APPLIED / STALE.
Decision is final, actor and timestamp are recorded. REJECTED requires a stable
reason (TIMING, DRIVER_OR_VEHICLE, RESOURCE_UNAVAILABLE, ROUTE_OR_SEQUENCE,
MISSING_CONTEXT); optional note ≤300 characters. A stale plan requires a new
generation and new decision. Acceptance/rejection alone never assign a task.

RecommendationExecution is a durable preview and unique global attemptId bound
to the original action, recommendation and preview hash. PREVIEWED → SUCCEEDED /
CANCELLED / STALE / FAILED. Terminal rows cannot be overwritten. A partial unique
index allows at most one SUCCEEDED execution per recommendation. Success requires
confirmation actor/time, result and unique audit FK. Cancel preserves ACCEPTED;
a new explicit attempt may be previewed afterward, without rewriting the plan.

## Snapshot and preview

Checks run on creation, acceptance, preview and execute. The snapshot contains
Budapest decision day; target assignment/status and relevant content; Job/order/
container binding/state; explicit next-day membership of the selected driver/truck; active driver
eligibility and truck capacity; aggregate global/truck load; ordered mixed source and
destination lists, pinned items, neighbour refs and run/load metadata (including
all members needed by existing run validation). Notes, audit timestamps, session
activity, unrelated plan memberships, resource display names and roles unrelated to driver eligibility are deliberately
excluded. The source and
target lists and capacity values are captured while the existing operational
EXCLUSIVE admission is held. No guarantee is claimed for unadapted external scripts.

Preview records old → new assignment, target planned and visible position, and
normalized positions in both affected lists. The server computes previewHash
from recommendation ID, attempt ID, immutable action, snapshot hash and summary.
The display labels are stored at preview time; a later resource name change alone
is irrelevant to eligibility. Date rollover, changed binding/resources/capacity/
order/sequence or invalid business rules produce STALE without task writes.

## Transaction and idempotency

Every service operation uses existing runOperationalWrite admission before
business reads. Prepare phase reuses #89's daily-plan and sorted dispatch locks,
then requester UserSession → User share locks. Recommendation → attempt row locks
follow. No new lock namespace, nested transaction or HTTP chaining exists.
The existing public reassignOverdueTask wrapper remains compatible and delegates
to prepare/apply on its original transaction. Fresh signed JWT, DB session,
active user, authVersion, expiration/revocation, OPERATOR mode and capability are
checked under locks, including immediately before application.

Apply, old/new list normalization, legacy assignment/order audits, execution
success, RECOMMENDATION_EXECUTED audit and APPLIED recommendation share one commit.
Failure before callback completion rolls back every write. Its optional failure
record is written in a separate admitted transaction, conditional on PREVIEWED;
it cannot overwrite success or cancellation. A failed attempt is terminal: there
is no automatic business retry. An operator may start a new confirmation flow.

A repeated successful attempt returns the saved result after fresh authorization,
without checking the pre-application snapshot or applying again. Other attempts
cannot execute an APPLIED recommendation. Cancel and execute serialize through
admission and row locks: cancel first prevents application; execute first returns
the saved success to a later cancel. No process-local promise is idempotency state.

An error after the transaction callback completed is UNKNOWN (HTTP 503), never
classified as a known rollback and never recorded as FAILED. No automatic retry.
Client must query result for the same recommendation/attempt. The result read
uses admission, so it waits for in-flight execution to settle and returns durable
SUCCEEDED, PREVIEWED or other terminal state. UNKNOWN is a response uncertainty,
not a fabricated persistent DB failure; resolved PREVIEWED means no application
committed. A new execute requires an explicit operator action.

## API

Under `/api/admin/operational-recommendations/:id/`:
- POST decision: `{decision: "ACCEPTED"}` or `{decision: "REJECTED", reason, note?}`.
- POST preview: `{attemptId}`.
- POST execute: `{attemptId, previewHash}`; no assignment/plan fields accepted.
- POST cancel: `{attemptId}`.
- GET recommendation: read the existing recommendation.
- GET result?attemptId=...: durable attempt/result lookup after a lost response.

Mutation routes use existing same-origin CSRF, operator RBAC and strict Zod
contracts. All services independently authorize the signed cookie against fresh
DB state; result replay is also protected. Responses are no-store. There is no
client generation route. No staging/production migration or deploy is performed.

## Evidence

Additive migration generated with Prisma migrate diff plus explicit SQL CHECKs,
immutable-content/terminal-state triggers and partial success uniqueness.
P63 migration/schema parity validates ordinary schema; integration tests exercise
custom SQL invariants, rollback faults, concurrency, UNKNOWN result resolution,
expiry/revocation and both target kinds. Existing #89 assignment/reorder/deadlock
regressions remain in the full gate. Local PostgreSQL startup is unavailable under
sandbox UID permissions; real PostgreSQL evidence comes from automatic CI only.

## Same-day type

`SAME_DAY_ASSIGNMENT_V1` shares this lifecycle (store, decide, preview, confirm, execute) but is
a separate type with `targetDay = decisionDay`, a mandatory re-evaluation at preview and a
10-minute confirmation age limit. See same-day-assignment-recommendation.md.
