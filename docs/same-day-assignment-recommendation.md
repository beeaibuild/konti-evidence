# Same-day assignment recommendation (SAME_DAY_ASSIGNMENT_V1)

A second, separate recommendation type next to the overdue/tomorrow one
(`OVERDUE_REASSIGNMENT_V1`, unchanged). It proposes where a task that is planned
for TODAY and has not started should go: which driver, which vehicle, and at which
position of that driver's order. The AI only suggests; an operator approves; the
existing preview/confirm/execute path applies it.

## What may be moved

- Only a TERVEZETT job or transport task whose planned date is Budapest's today.
- The task may be unassigned or already assigned to someone else.
- FOLYAMATBAN tasks and tasks bound to a dispatch run are never moved; FOLYAMATBAN
  tasks stay pinned in front of the planned ones.
- An explicit daily plan for today is required. Driver and vehicle must be in it.
- Capacity, container and order state rules are the same as in the overdue flow
  (`evaluateCandidateDay`), with the "later than the original date" rule replaced
  by the explicit today check. Placing a task exactly where it already is is refused.

## Where the starting point of a vehicle comes from

For every (driver, vehicle, gap) candidate the start of the gap is chosen in this order:

1. The end point of the planned task right before the gap.
2. The end point of the task in progress, when nothing planned precedes the gap.
3. The newest trusted GPS fix of the vehicle, only when nothing is committed before the gap.
4. Otherwise unknown.

A vehicle that is working is therefore priced from where its work ends, not from where
it is now: a nearby vehicle is not necessarily free. A GPS fix is trusted only when it is
the newest point of a still-authorised open shift, not simulated, accurate to 100 m or
better, and not older than 10 minutes (a timestamp more than 2 minutes in the future is
rejected). GPS is read only when `ENABLE_GPS_TRACKING` is on. Raw coordinates are never
stored in the generation context, the recommendation, the snapshot or the audit log; only
derived values (source, data age in seconds) are kept. The 7-day GPS point retention is
unchanged.

**Vehicle continuity.** The day's items are loaded per driver, but every item keeps its
`truckId`. An earlier task's end point is the start of a vehicle only when that task is on
the same vehicle; the route leg to the next task is priced only when the next task is on
the same vehicle too. Otherwise (another vehicle, or no vehicle recorded) the start or the
next leg is unverified: GPS does not stand in for it, the estimate is `NOT_EVALUABLE` with
reason `VEHICLE_CONTINUITY_UNKNOWN`, and the otherwise lawful candidate stays selectable.

A driver with an open GPS shift is offered only with the shift's vehicle, and a vehicle on
an open shift only with the shift's driver (the shift is the real pairing). Without a shift
every pairing allowed by the daily plan is offered and marked `PLAN_ONLY`.

## The travel estimate, and what it is not

The estimate is the detour of the insertion: start -> new task -> next task, minus
start -> next task (for the last position just start -> new task), as road distance and
road time between known coordinates **with current Google traffic**. Straight-line distance is used
only to choose the best 5 candidates for the road-route calls; it is never shown and never
converted into minutes. The estimate is not an arrival time and contains no loading,
unloading or remaining working time. Nothing in the product claims saved time.

Every leg uses departure at request time (Google's default), including legs after earlier
work. The result is a current-traffic comparison, **not a scheduled route simulation**:
we do not invent loading times or later departure times. Actual traffic when the vehicle
reaches a later gap may differ. `traffic=CURRENT` and `evaluatedAt` are retained through
candidate enrichment, AI facts, saved recommendation and preview; the UI shows the snapshot
time. Legacy estimates retain `NOT_CONSIDERED`. A change of traffic basis at preview requires
new operator approval. There is no persistent route-result cache.

Traffic-aware Google calls have a higher billing rate. Depot pricing/distance and its
route preview retain `TRAFFIC_UNAWARE`; this change requires no APK rebuild or DB migration.
Live provider acceptance remains pending deployment after the physical midnight test.

When a figure cannot be computed the candidate stays selectable and is labelled "not
evaluable" with the reason (`NO_TARGET_POINT`, `NO_START_POINT`, `NO_NEIGHBOUR_POINT`,
`NOT_SHORTLISTED`, `ROUTE_UNAVAILABLE`, `PROVIDER_UNAVAILABLE`). No depot or other
substitute point is invented. A missing GPS fix or a missing coordinate never blocks an
otherwise lawful assignment, and no distance or time statement is built on such data.

The point-to-point route is available only with `DISTANCE_PROVIDER=GOOGLE_ROUTES`
(`routeBetweenPoints`, field mask `routes.distanceMeters,routes.duration`, `TRAFFIC_AWARE`). The MOCK provider never invents a route, so with MOCK every candidate is "not
evaluable" (`PROVIDER_UNAVAILABLE`). Addresses need coordinates (`Address.lat/lng`); the
code does not geocode and does not guess them.

## Generation

Candidates are enumerated deterministically (driver ID, vehicle ID, gap order) as in the
overdue flow, the model returns only a candidate ID or "no recommendation". Differences:

- External route calls run between two short database transactions, never inside one.
- The claim step re-reads the structure; if it moved while routes were priced, the result
  is STALE. Staleness is judged on a structural hash that excludes position values, so a
  new GPS point alone does not invalidate a request.
- Versions: prompt `same-day-choice/3`, rule `same-day-candidates/3`, gps `same-day-gps/1`.
- No historical feedback examples are sent for this type (they were built for the overdue
  type and would not be comparable).
- Provenance stores, next to the chosen candidate, a rule-based reference computed twice:
  `reference.withGps` (least estimated added minutes with the real start points) and
  `reference.withoutGps` (the same rule with GPS-started candidates made not evaluable).
  The reference is a deterministic yardstick, not the same AI without GPS.

## Preview, re-evaluation and execution

Opening the confirmation re-evaluates the approved variant outside any transaction, then
the preview runs in its own transaction. The approved variant is never swapped silently.
A new recommendation (and a new explicit approval) is required when:

- the approved variant's road estimate is no longer available (`BASIS_LOST`);
- the same variant became materially slower: at least 5 minutes **and** at least 20 percent
  of the approved estimate (`ESTIMATE_WORSE`);
- the rule reference now points to a different variant that is materially better under the
  same thresholds and was not already the reference at generation time (`BETTER_VARIANT`);
- the re-evaluation cannot run at all (`EVALUATION_UNAVAILABLE`).

Small changes (for example +3 minutes) keep the same confirmation executable. A change of
car, driver or insertion point is always a new recommendation. Each such outcome is written
to the audit log as `RECOMMENDATION_REVALUATED` (no coordinates). The generic preview entry
refuses a same-day row without a fresh evaluation (`INVALID_STATE`), so the revaluation
cannot be bypassed. A previewed confirmation older than 10 minutes is closed
(execution CANCELLED, `expired: true`) at execute time and must be opened again. Execution
re-validates the stored snapshot inside its transaction, exactly as for the overdue type.

**Pairing at execution.** The shift pairing above is also enforced by the shared
prepare/execute check, after the daily-plan lock: if the open shift's driver–vehicle
pairing no longer allows the assignment, nothing is modified, no execution audit entry is
written and the execution is STALE. Without GPS (or with `ENABLE_GPS_TRACKING` off) the
check is a no-op. Opening a shift (`startGpsShift`) and executing a recommendation both run
through `runOperationalWrite`, which takes the same EXCLUSIVE application-scope admission
lock. A shift cannot therefore be opened while an execution is in progress, or the other
way round: the pairing read at execution is consistent with every shift opened the normal
way. No additional lock is needed or taken.

**Estimate bound to the base.** The re-evaluation returns a digest of the state it read:
the structural hash (day, target, per candidate: id, variant key, snapshot hash, pairing
and start source) plus a fingerprint of the stored coordinates the estimate is priced from
(the target's entry/exit points, the next task's point and a start taken from a task).
Live GPS points are not part of either; the fingerprint is a hash held in memory only, so
no raw coordinate is stored in the recommendation. Inside the preview transaction, before
the revision is accepted, the structure is rebuilt and compared. On mismatch no preview is
created from the old estimate: the evaluation (and the route request, always outside any
transaction) is repeated once; if the base changed again the recommendation is closed
STALE with reason `BASIS_CHANGED`.

## Storage

Migration `20261014200000_same_day_recommendation` only rewrites two CHECK constraints
(`recommendation_dates`, `recommendation_action_target`): same-day rows must have
`targetDay = decisionDay`, overdue rows keep `targetDay = decisionDay + 1`. No table,
column or enum changes; the immutability triggers are untouched.

## UI and metrics

The daily-operations page shows an "AI-ajánlás mára" card only when the selected date is
Budapest's today. The card shows the start source with the GPS data age, the estimate or the
"not evaluable" reason, and the fresh estimate in the confirmation. The metrics page has a
separate same-day section; the established overdue figures are unchanged. The GPS counters
are descriptive (how often a GPS start existed and whether GPS changed the rule reference and
whether the model followed it). They do not prove an improvement.

## What the tests prove, and what they do not

Controlled route responses (including a simulated river-crossing penalty) prove that the
pipeline uses the GPS start, ignores untrusted fixes, handles busy vehicles and missing data,
re-evaluates and binds execution to the approved variant. They do not prove that the real AI
chooses better, nor that real road figures look like the simulated ones. Verifying the real
provider, real coordinates and a real tablet is a manual staging task.

## Prompt version 2: missing travel estimates

The model is told that supplied candidates passed the application's eligibility checks.
NOT_EVALUABLE describes the travel estimate, not eligibility. Without route or GPS data,
it may recommend a placement using supplied loads and order for human approval. It must
not invent travel savings, physical compatibility or immediate availability. Empty lists
and concrete conflicts still permit refusal. Validation and execution checks are unchanged.

A live gpt-4.1-mini comparison on 2026-10-15 replayed two saved staging inputs three times
per prompt. Version 1 declined all six; the clarified prompt selected valid IDs in all six.
Both prompts chose the lower travel estimate in three synthetic comparisons and declined
all three empty-list controls. These small checks demonstrate the fallback contract, not
better dispatch decisions or statistical reliability. No model change or migration is needed.

## Internal transport pickup coordinates (2026-10-16)

New internal transports geocode both complete structured addresses before entering the write transaction.
Optional originLat/originLng are persisted separately from the destination. Missing or failed geocoding stays unknown;
existing rows are not backfilled and a label such as Telephely never substitutes a depot coordinate.
The immutable pickup address keeps its coordinates when cargo, note or destination is edited.

The road estimate includes start -> pickup -> destination -> next, minus start -> next.
The pickup-to-destination leg is included even at the end of the schedule; a failed leg makes the whole estimate unavailable.
Single-address order jobs need no extra leg. Neighbouring internal transports are approached at their pickup address.
Candidate rule version is same-day-candidates/3. Stored-coordinate verification already includes entry/exit points.

A real route test still needs a trusted GPS start or the known end of an earlier task on the same vehicle.
An empty vehicle schedule never assumes the vehicle is at the depot. Traffic is a request-time snapshot, not a forecast for the later gap.
