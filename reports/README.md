# Reports

Three historical evaluation reports referenced in [EVIDENCE.md](../EVIDENCE.md), section 3. All runs used the earlier
20-case benchmark (`konti-viber-v1`), `gpt-4.1-mini`, temperature 0, on a non-production staging environment.

| File | Content |
| --- | --- |
| [2026-09-10-baseline-7398002.md](2026-09-10-baseline-7398002.md) | Baseline real-provider run: 40.84 points, 19 runner-critical cases, human review separating scorer false positives from real model errors (English). |
| [2026-09-11-postfix-975e45a9.md](2026-09-11-postfix-975e45a9.md) | Run after the scorer/eval fix: 50.63 points, 0 critical; what the fix changed and what it did not (English). |
| [2026-09-11-baseline-variability-975e45a9.md](2026-09-11-baseline-variability-975e45a9.md) | Seven runs on the unchanged configuration: mean 48.77, SD 2.22 points; case-level variability, unstable fields, and the embedded structured outputs per run (Hungarian, with English field names). |

## Privacy modification

These are **privacy-modified copies**, not byte-exact copies of the original reports. Task, partner and person identifiers
from the September fixture version (place names and driver names) were replaced with the pseudonymised aliases used in the
current benchmark fixtures of the private repository (for example `task-harsalja`, `user-tibi`, `user-bence`), consistently
across all three reports, including the embedded JSON. A one-line notice was added at the top of each file. Nothing else
was changed: scores, metrics, flags, run identifiers, commit SHAs and dates are as in the originals.

The SHA-256 values inside the reports refer to the original raw JSON reports, which were written to an ephemeral staging
filesystem and are no longer available. `SHA256SUMS` in this folder verifies the published copies only.

The reports contain the structured model outputs for the seven-run study (merged into identical variants; natural-language
summaries omitted). They do not contain complete raw provider responses.
