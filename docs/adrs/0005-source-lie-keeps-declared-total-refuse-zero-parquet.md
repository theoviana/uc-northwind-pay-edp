---
adr: "0005"
status: accepted
date: 2026-08-25
ground: brownfield
converge_pass: 2
spec_ref: "R-1, R-2, R-3"
supersedes: ""
superseded_by: ""
deciders: "Marina Alves (kick-off D3, no-repair rule); Structure pass agent (this record)"
---

# 0005 — source lie keeps the declared total; refuse; zero Parquet rows

## Context

R-2 already requires refusing, never repairing, a control mismatch, and
R-3 already requires that "a kept source lie" is one of the three valid
terminal states. Neither requirement states the resulting row count at
the landing layer. The legacy plant's own already-observed behavior for
this exact fixture is the fact to ground, and the same rule extends by
symmetry to the second plant's Parquet landing.

## Decision

For a batch whose declared control total disagrees with the
independently computed total (the Type `01` fixture: declared `173.44`,
computed `173.45`), the declaration stays unedited and the batch is
refused. On whichever plant observes it, the output row count for that
batch is zero at the layer that would otherwise carry business rows —
zero sanitized CSV rows on the legacy plant, and by the same rule, zero
Parquet rows on the second plant's landing.

## Rejected reading

Could have read as: land a diagnostic row into Parquet (flagged
`rejected` but still present), so the refusal is visible to downstream
Bronze/Silver/Gold queries without a separate side-channel. Rejected —
the legacy plant's own observed fixture shows zero rows, not a
flagged row (`csv_produced: false`), and the drop's refusal rule reads
"no CSV, no business rows," a hard zero, not a soft flag; a landed row
of any kind would be a form of repair by inclusion.

## Evidence

**Authoritative** (the judge):
`contracts/types/01-card-settlement/main/expected-df-source-001-finding.yaml`:
`expected_status: quarantined`, `csv_produced: false`,
`postgres_business_mutation: false`, `declared_net_amount: "173.44"`,
`computed_net_amount: "173.45"`.

Corroborating color only (inbound, not authoritative):
`spec/type-01-card-settlement/expected/df-source-001.finding.yaml`
mirrors the same fixture verbatim as of this drop.

```sh
grep -n "csv_produced\|postgres_business_mutation" \
  contracts/types/01-card-settlement/main/expected-df-source-001-finding.yaml
```

observed output:
`contracts/types/01-card-settlement/main/expected-df-source-001-finding.yaml:11:
csv_produced: false` /
`contracts/types/01-card-settlement/main/expected-df-source-001-finding.yaml:12:
postgres_business_mutation: false`.

## Consequences

Pass 5 task-specs and evals for the second plant's equivalent of
`df-source-001` must assert zero Parquet rows written for that batch
under `modern/landing/`, mirroring the legacy assertion of zero
sanitized CSV rows and zero business mutation. An eval that only checks
"batch marked refused" without also asserting zero landed rows would
under-specify this fact.

Re-verify when: a source-lie fixture's expected finding changes
`csv_produced` away from `false`.
