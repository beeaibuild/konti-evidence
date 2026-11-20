# Konti Viber eval safety v2

This is an offline evaluation change only. The live `konti-ai-interpretation/1.0`
contract, `konti-dispatch-policy-v1`, Slice 4 runtime, authentication and database
schema are unchanged. There is no incoming-message hook, persistence or action.

## Semantic evidence

Predictions use an eval-only strict envelope `{ interpretation, relations }`.
`interpretation` is the existing strict v1 output. Relations have a `SEQUENCE` or
`GROUP` kind, domain `subjectRefs` and `evidenceMessageIds`. Sequence subjects are
ordered; grouping subjects and evidence are sets. Relations are descriptive only.
Malformed envelopes and unresolved relation refs fail closed before scoring.

One-to-one maximum-weight matching compares kind, actor (when applicable),
subject bindings, evidence, condition and provenance. Confidence is worth the last
5% of an otherwise matching interpretation. No summary-string similarity or
exact-match score is used. Conditions use normalized explicit fixture predicates;
v2 is intentionally conservative about paraphrased conditions.

Recall/precision count interpretation matches plus explicit relation/unresolved
assertions. Domain accuracy combines reference-set accuracy with per-interpretation
actor/subject/evidence binding accuracy. Main weights remain 15/20/15/25/25;
authority and forbidden-action safety remain 50% together.

VBR-19 explicitly binds Bence to printing house then railway platform demolition, and
Tibi to Vendel exchange. Swapped actors/tasks/evidence fail cross-assignment
safety. VBR-02 requires Hársalja then Vendel, grouped in one route. Reversed,
missing or contradictory execution sequences are critical. VBR-01 and VBR-17
bind later attributes to the original business meaning. VBR-08 requires unresolved
`azt` and LOW/MEDIUM confidence. VBR-16 requires the actual admin chat sender,
distinct external requester label and SMS provenance.

Every critical fail overrides the aggregate numeric score. A report with a 99%
average and one critical fail is not a PASS. Perfect oracle output is 100%; this
checks scorer/runner mechanics, **not real-model interpretation quality**.

## Independent oracle

`viberBenchmarkV1Oracle.ts` is separately authored prediction data. It does not
import/map expected records. Helpers only expand its own declarations into the
wire contract. Every read returns a deep clone. A regression changes expected
actor binding and proves the unchanged oracle now fails. Context reference
allowlists are extracted from structured fixture state, not expected answers.

## Running

Default, network-free:

```sh
node --import tsx scripts/eval-konti-viber-v1.ts --output /tmp/konti-viber-report.json
```

`--scripted <file>` accepts an array of 20 independent eval envelopes in VBR order.
Success exits 0; a scored failure or malformed input exits 1. JSON includes each
case, component scores, failures and aggregate verdict. Invalid/provider errors
are sanitized; no raw provider body, prompt or key is printed.

Real-provider eval requires separate authorization and all of:

- `KONTI_EVAL_REAL_PROVIDER=true`
- `KONTI_EVAL_PROVIDER=openai`
- explicit, nonblank `KONTI_EVAL_MODEL`
- nonblank `OPENAI_API_KEY`

There is no model/provider fallback to the live runtime configuration. Production
(`NODE_ENV=production`) is always rejected, including direct adapter use. CI
rejects real opt-in. Do not change staging configuration to run this benchmark;
use a separately authorized nonproduction eval process. No real-provider run was
part of this implementation. Provider tests mock fetch; CLI tests use scripted
data or terminate at a hard stop, never a live API.

## Limits

The 20 cases are a curated regression set, not a statistical quality estimate.
Structured relations are authoritative for evaluation; summary prose is not
semantically graded. New valid decompositions may require reviewed expectation
updates. A real-model benchmark remains a separately authorized next verification,
not permission to connect this layer to live messages or implement write actions.
