# Weekly AI recommendation choice — V1

The weekly overview keeps its existing read-only RULE suggestions and manual
rescheduling flow. A separately labelled AI card lists overdue TERVEZETT Jobs and
TransportTasks (including those dismissed in the legacy RULE flow). Mount/refresh
only reads the latest generation. Only the explicit request button generates.

## Candidates and privacy

The server enumerates tomorrow's Europe/Budapest explicit plan, sorted by driver
ID then truck ID, then each gap in the existing common dispatch list. Every
candidate is validated by #89 prepare, without applying it. The existing container,
capacity, plan-membership, run-integrity, pinned-prefix and stable neighbour rules
are the single business authority. No inferred driver/truck pairing is invented.
No source/destination task's relative order is changed by candidate construction.

The complete enumeration is bounded at 256 potential gaps across all pairs; if it
exceeds this limit, the whole request is declined with CONTEXT_LIMIT, with no model
call. The same applies if the complete model facts exceed the configured input
budget. There is no first-driver, first-truck, or prefix truncation. Eligible IDs
c0001.. are deterministic within the request; full action references stay server-side.

The provider sees target type and city/postcode areas; anonymous driver/vehicle
labels, their load/capacity; adjacent planned task types, sequences and areas;
visible insertion position, and known limitations. Personal names, plates, emails,
phones, raw database IDs, street addresses, notes, container/customer labels and
raw cargo text are omitted. All remaining text fields are untrusted JSON data,
not instructions. This (overdue) type is appropriate placement, not route/ETA/weight/shift or
physical-compatibility optimization. The same-day type has its own, clearly labelled
road estimate without traffic; see same-day-assignment-recommendation.md. A first planned gap is after all pinned tasks.

## Provider and persistence

The existing getKontiAiConfig/createProvider adapter is used. No new model selector
or secret is added. KONTI_AI_MODEL is the actual recorded model; the prompt and
rule versions are overdue-choice/2 and overdue-candidates/1. Provider requests
have no tools and maxRetries=0, regardless of the read-chat retry configuration.
Only strict Zod {candidateId:string} or {noRecommendation:true} is accepted, with
complete stop finish, no tool calls and an ID in the original candidate set.
Invalid output is INVALID_OUTPUT, never RULE. Fake provider is non-production and
saved MODEL provenance isTest=true. Production paid provider is never used in tests.

The existing AiProviderAttempt table requires chat Message/Conversation foreign
keys and cannot own a weekly request. The small RecommendationGeneration table
therefore uses the same existing AiProviderAttemptStatus pattern, scoped to this
feature. It records a unique UUID, target hash, requester/session/authVersion,
immutable original context and fingerprint, actual provider/model/versions, result
and optional unique recommendation link. There is no general generation framework.

Claim occurs under existing EXCLUSIVE operational admission, before any provider
call. Same request/target/user replays never invoke another provider. Conflicting
identity fails. STARTED replay is reported UNKNOWN; after process/network failure
it cannot automatically call again. Valid output is saved RESULT_READY; explicit
same-request POST can finalize it without another provider call. Reads never create
recommendations. UNKNOWN is terminal. A new explicit request uses a new UUID.

The provider network stream is awaited outside DB transactions. Before start and
before saving/finalization, authority is checked fresh. Finalization regenerates
the complete candidate facts and fingerprints under admission. Any original
candidate-state/fact change or Budapest rollover results in STALE, without storing
an OperationalRecommendation. The selected original snapshot hash is also checked
inside the #90 store helper in that same transaction. The helper's standalone
wrapper and previous callers remain compatible; no nested transaction is opened.
New MODEL snapshots use v2, including target Job and neighboring task city/postcode
areas, so changes after generation also invalidate decision/preview/execute. Existing
v1 records keep their original validation semantics and are never rewritten.

Operational advisory locks are acquired up front: tomorrow daily lock, all source
and destination groups in existing lexical driver/date order, then requester
UserSession -> User locks. Candidate prepares only reacquire held locks. Read-only
result endpoints have admission and authority locks; they do not need plan locks.
The guarantee remains bounded to adapted writers, with old writers drained; it
is not a repository-wide or unadapted-script lock guarantee.

## UI lifecycle

Accept records #90 ACCEPTED only, then opens a server preview. Execute sends only
attemptId and previewHash to #90. Cancel preserves ACCEPTED. Reopening after an
explicit cancel creates a new attempt; lost responses and UNKNOWN query the
original attempt. A resolved PREVIEWED result requires another explicit Execute,
never auto-execute. SessionStorage retains request/attempt UUIDs for tab refresh;
the durable DB owns all results, with fresh authorization on replay. Reopening is
blocked until mount-time attempt recovery finishes. A new explicit attempt clears
the old preview before requesting the server preview; if that response is lost,
an explicit preview retry keeps the newly saved attempt UUID. An unresolved
execution continues to recover its original attempt, never an automatic new one. No IDs, hashes,
raw response or snapshots are displayed. Why this shows saved facts, not invented
model reasoning and never another model call. Rejection requires one of the five
stable reasons and <=300 optional characters. Success refreshes the overview.

HU is default; locale=en supplies English AI cards. The existing weekly/RULE page
retains its HU navigation; this scope does not translate the entire admin app.
Dark classes and wrapping button groups cover mobile and 1024x600 landscape.

## Validation

Integration: eligible choices and all resource pairs, output failures/decline,
provider-time target/area/plan/capacity/day and authority changes, duplicate claims,
no-plan/no-candidate no-call, privacy, uncertain outcomes, durable replay and the
separate #90 confirmation. Existing #89/#90, manual assignment/reorder and RULE
regressions run in the full gate. UI E2E: short card and on-demand facts, required
rejection reason, read-only refresh, accept/cancel, lost response, UNKNOWN, STALE,
double clicks, English mobile/tablet. P63 includes these specs and report evidence.
No deployment, live migration, workflow_dispatch or manual rerun is part of author validation.

Metrics and previous structured feedback: see [operational-recommendation-metrics.md](operational-recommendation-metrics.md).
