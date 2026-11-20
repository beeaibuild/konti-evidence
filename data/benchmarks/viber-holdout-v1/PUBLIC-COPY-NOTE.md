# Public copy of the 18-case held-out set — privacy-modified

This folder is a **privacy-modified copy** of the frozen held-out package prepared on 8 November 2026.
It is **not** a byte-exact publication of the original files. The original package stays unchanged in the private
repository; its original SHA-256 list continues to verify the original files only. The `SHA256SUMS` file in this folder
verifies the published, modified copies.

The source texts were already pseudonymised before this step. For the public copy, a second, conservative pass was applied:

- **Names, companies and addresses** were replaced again with new aliases, consistently across `REVIEW.md`,
  `selection.json`, `source-ledger.json` and `validation.json` (messages, manual expectations and metadata alike).
  Driver aliases were also replaced (three drivers; the dispatcher role label is kept). Spelling errors, accents,
  punctuation, suffix forms and quantities were kept where possible, because they are part of the interpretation task.
- **Dates** were converted to relative days: D+0 (a Friday), D+3, D+4 and D+5 for the 18 cases; D+1–D+2 (weekend) and D+6–D+7
  appear only in excluded source records. Order and spacing are preserved; message clock times are unchanged.
  Process timestamps (freeze and selection time, 7–8 November 2026) are kept.
- **One private detail** was generalised: in H06 a driver's family member was replaced by "an acquaintance"
  ("lerobbant egy ismerősöm"). The interpretation task is unchanged: a breakdown that is not proven to be a company truck fault.
- A container number in H15 was changed (still a three-digit number with the same grammatical form).

Not included: the reverse pseudonym map, raw screenshots and source text, `REVIEW.html` (a duplicate of `REVIEW.md`)
and `APPROVAL.md` (it refers to the original, unmodified files). No saved model responses from the three held-out runs
are included in this repository, so no model output was modified.

Hash fields inside the JSON files (`sourceSha256`, `freezeHash`, `definitionHash`, screenshot hashes) and the byte and
character ranges in `source-ledger.json` refer to the **original private source**; they are kept as provenance identifiers
and cannot be re-checked against these modified files. `freeze.json` is unchanged.

These 18 cases became development data after the three runs (6/18, 5/18, 5/18; 4 stable, 2 fluctuating, 12 FAIL).
Any future claim requires a new freeze and a new, unpublished held-out set, which stays private until its measurement is closed.
