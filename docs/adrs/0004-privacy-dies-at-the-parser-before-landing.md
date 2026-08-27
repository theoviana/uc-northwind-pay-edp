---
adr: "0004"
status: accepted
date: 2026-08-25
ground: brownfield
converge_pass: 2
spec_ref: "R-11"
supersedes: ""
superseded_by: ""
deciders: "Priya Shah (privacy-boundary decision, 2026-06-16); Structure pass agent (this record)"
---

# 0004 — privacy dies at the parser, before landing

## Context

Privacy is core to both the legacy plant and the drop's own kick-off
(D4, Priya: "Privacy is finished before any CSV") and the dedicated
2026-06-16 privacy-boundary meeting — but at the time this ADR was
first drafted, `docs/tech-spec-type-01-card-settlement.md`'s
Requirements section (R-1–R-5, W-1–W-7) carried no explicit privacy
requirement. That was a real gap in Pass 1's coverage, not evidence
that privacy was out of scope. This ADR grounded the fact directly
from the drop, and the gap it flagged has since been closed: the
tech-spec now carries `R-11` (privacy dies at the parser), added at
this Consensus pass and cited in this record's `spec_ref`.

## Decision

Restricted values — PAN, CPF, CNPJ, account numbers, and any other
value a type's privacy policy names — must not survive past each
plant's own first transform of the raw record. On the legacy plant that
boundary is Java; on the second plant, the equivalent boundary is its
own parser, before any write to `modern/landing/` or any layer beyond
it.

## Rejected reading

Could have read as: the rule only binds the legacy loader (D4 says "the
loader must not see a PAN"), so the second plant may defer
tokenization to a later Bronze/Silver/Gold step instead of its parser.
Rejected — the privacy-boundary meeting's own Executive Summary
extends the rule past the loader explicitly: restricted values "must
die at the modern parser **before any Parquet or Gold**," not merely
before a downstream loader.

## Evidence

`spec/estate/meetings/2026-06-16-privacy-boundary.md`, Executive
Summary: "Restricted values die at Java on the live line and must die
at the modern parser before any Parquet or Gold." `spec/estate/policies/privacy.md`,
"Must not exist after sanitize": those values in the clear "in any CSV,
Parquet, log, evidence packet, ticket, or warehouse table, unless a
type policy names an approved transform."

```sh
grep -n "modern parser\|before any Parquet" \
  spec/estate/meetings/2026-06-16-privacy-boundary.md
```

observed output: `spec/estate/meetings/2026-06-16-privacy-boundary.md:13:
Restricted values die at Java on the live line and must die at the
modern parser before any Parquet or Gold.`

## Consequences

Every later pass (seams, tasks, evals) that touches the second plant's
parser must treat privacy transformation as inside that parser's own
boundary, not deferred to a later Bronze/Silver/Gold step; a task-spec
for the parser with no privacy eval is incomplete. Pass 5 task-specs
for `leg-03-parser` cite `R-11` in their `spec_ref`, closing the
traceability chain this ADR originally flagged as broken.

Re-verify when: a type's `privacy.yaml` or the privacy-boundary
decision is revised, or `R-11` is amended or superseded.
