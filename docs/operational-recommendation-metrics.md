# Recommendation measurement and previous feedback — V1

Telemetry remains in `/master-admin/telemetria?nap=7|30|90`. The page and read
service both verify current master-admin scope before any business queries.
Opening it performs only reads, never generation or recommendation writes.

## Cohort and rates

The period includes the last N Budapest calendar days (today included), starting
at actual Budapest midnight and ending at the captured current instant. Weeks
start Monday in Budapest. All recommendation metrics use the created-at cohort,
not the current execution/decision date. The order change percentage separately
compares the last N complete Budapest calendar days with the preceding N complete
days, both [start, end); today is excluded. Bounds use date-only calendar shifts
before conversion to Budapest midnight, so DST days remain whole calendar days.
The order count/source cards retain the selected current cohort; their change
caption explicitly identifies the separate complete-day comparison. Recommendation IDs deduplicate samples;
retry attempts never multiply applications. MODEL + non-test + non-fake is the
main cohort. RULE and test/fake counts are separately labelled.

Immutable decidedAt/rejectionReason identify human acceptance/rejection even
when ACCEPTED later becomes APPLIED or STALE. PENDING and decisionless STALE are
unanswered. No modified-plan decision exists in the current contract; we add no
such UI and do not infer it from manual reassignment.

Acceptance = unchanged acceptance / answered MODEL recommendations. Application
= recommendations with SUCCEEDED / recommendations with a stored confirmedAt.
Cancelled/failed/stale/previewed counts are recommendation-level and may overlap
across attempts. UNKNOWN is a response uncertainty, not durable state in #90;
its execution count is explicitly unavailable rather than fabricated. Stored
confirmations cannot reconstruct confirmations that #90 did not persist.

Read-only RepeatableRead loads the cohort/executions/application audits, targets,
and relevant target/group audits in batches. The immutable destination snapshot
and applied insertion identify the original planned neighbours and their side of
the target. Their current states and assignment/reschedule audits are included
in the same batched queries. Leaving and re-entering on the opposite side counts
as modification even when the target's own fields never change. A same-side
return or neighbour truck-only change does not. Missing snapshots, neighbour
states or inconsistent audit chains cannot prove unchanged attribution. No per-recommendation SQL query.
Applied results must agree with the bound RECOMMENDATION_EXECUTED audit and plan.
Assignment/reschedule before/after and dispatch-group before/after determine
post-application day/driver/truck/relevant relative-order changes. Merely removing
completed neighbours may renumber without changing relevant order. A changed
current plan without adequate audit is unknown. An audited later change, even if
reverted, remains “Utólag módosított terv”. Audits cannot prove the absence of
unadapted external writes; the existing bounded writer/audit coverage still applies.

Completion requires target completedAt and a completion audit after application.
Budapest completion day is compared with the applied date-only plan. Failure and
cancellation use the business audit instant, never the planned day. Missing targets,
application/outcome audits or inconsistent data produce “Nem megállapítható”.
Actual task counts include modified plans, but attribution and completion-day
comparison exclude modified/unknown plans. Completion rate = unchanged completed /
unchanged applied targets with known outcomes (including open). Every rate shows
numerator/denominator, with “Nincs adat” at zero denominator. Weekly/version tables
show observed acceptance/completion rates and cohort sizes, not proven improvement.
The optional 80% is labelled “használhatósági cél”.

## Previous feedback selection

Only a new generation claim selects feedback, under existing operational admission.
The existing immutable RecommendationGeneration.context stores the full payload,
feedback version `overdue-feedback/2`, and selected recommendation references.
OperationalRecommendation.versions.feedback and provenance.feedback retain the
version and references on successful finalization. No migration or dependency.
Candidate contextHash still fingerprints the current operational candidates;
feedback is immutable request context, not eligibility. Finalization recomputes
candidates, not feedback. Same requestId recovery bypasses selection and provider.
Legacy requests without feedback remain readable/finalizable.

Deterministic selection: real MODEL decisions within the last 90 elapsed days,
same OVERDUE_REASSIGNMENT_V1 and target kind/type, with a matching destination
postcode or nonempty city. Score each example against the most similar single
current permitted placement: destination postcode 8 / city 4; origin postcode 4 /
city 2 when both known; matching Budapest calendar weekday 2 when stored;
matching driver-load bucket 2, vehicle-load bucket 2,
capacity-utilization bucket 3, insertion-position bucket 1; each matching adjacent
task type 2 and area 2 (matching empty boundary 2). Load/position buckets are
0, 1–3, 4+; utilization buckets are below 50%, below 80%, 80%+.
Rank by score descending, then decidedAt descending, then lexical ID descending;
take at most five. No matching area/type or usable current placement means empty
feedback. Human decision and allowlisted rejection code are advisory output, not
score bonuses. Unknown reason codes are omitted by excluding that example.
The immutable facts do not store usable time-of-day, cargo/container compatibility or
stable anonymous resource pairs, so those fields are not inferred or added.
This is operational similarity, not route optimization or a learned preference.

Allowlisted example data: original target type and structured origin/destination
areas; selected load/capacity/insertion position and structured adjacent task facts;
human decision and rejection reason code. No notes, chats, contacts, raw IDs,
names or plates. No modified-decision plan is currently stored. Database references
stay internal and are not sent to the provider. Historic anonymous resource labels
are also omitted to avoid confusing them with current candidate resources.

The provider receives the unchanged current candidates plus feedback version/examples.
Prompt version is `overdue-choice/2`; candidate rule stays `overdue-candidates/1`.
Feedback is advisory untrusted JSON, cannot authorize/ban candidates, and does not
change the strict candidateId/noRecommendation contract. The existing input budget
covers the entire combined payload. UI says “Korábbi visszajelzések felhasználása”.
No fine-tuning, automatic training, counterfactual comparison or improvement claim.

## Author validation

Targeted unit tests cover cohort separation, decisions, retries/cancel, percentages,
Budapest midnight/DST, version/week counts, outcomes and modified/uncertain plans,
rendered labels, and denied access before reads. Real PostgreSQL integration tests
exercise fake-provider receipt, <=5 examples, persistent references/privacy and
same-request replay; real application/cancel/retry and task outcome audits.
Local PostgreSQL cannot start under this runtime's available UID permissions;
DB evidence comes from the automatic P63 isolated PostgreSQL service. No live DB,
paid model calls, workflow_dispatch, manual rerun, merge or deploy is used.
This is author verification; independent review remains outstanding.

## Same-day section

Same-day recommendations are excluded from the figures above and reported in their own
`sameDay` section, including descriptive GPS counters. The counters do not prove that GPS improved
decisions. See same-day-assignment-recommendation.md.
