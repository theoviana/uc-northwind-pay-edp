---
adr: "0003"
status: accepted
date: 2026-08-25
ground: brownfield
converge_pass: 2
spec_ref: "R-1, R-2, R-3"
supersedes: ""
superseded_by: ""
deciders: "Rafael Costa (approved D2, 2026-06-09); Structure pass agent (this record)"
---

# 0003 — money is exact decimal, never float

## Context

R-1 (independently recompute the control total) and R-2 (refuse at
zero tolerance when declared ≠ computed) only mean what they say if the
arithmetic behind "recompute" and "≠" is exact — a binary-float sum
can silently disagree with a decimal sum by less than a cent and either
manufacture a false mismatch or hide a real one. The legacy plant and
the drop both already fix money's representation; Structure grounds
that fact so it is not left implicit going into Pass 3.

## Decision

Every live type's **money** field is exact decimal at scale 2. The
second plant's arithmetic never uses binary floating point for a money
value — this matches the legacy plant's own representation
(`numeric(18,2)` / `decimal(18,2)` columns throughout) and the drop's
own rounding policy. This decision is scoped to money fields only —
see Evidence for a non-money field that uses a different scale.

## Rejected reading

Could have read as: a native binary float (e.g. a language's default
`double`) is acceptable for Type `01` specifically, since its only
values are a two-record sum (`173.45`) with no percentage rounding
involved (unlike Type `05`'s `HALF_UP` case). Rejected — D2 states
"Exact Decimal. No float money" with no per-type carve-out, and the
legacy schema itself already uses `numeric(18,2)`/`decimal(18,2)` on
every money column observed, not only Type `05`'s.

## Evidence

`spec/estate/meetings/2026-06-09-file-decomposition.md`, D2: "Exact
Decimal. No float money" (owner Rafael, Approved).
`spec/estate/policies/rounding-and-controls.md`: "Money is exact
decimal... **Do not** use binary float."
`legacy/postgres/migrations/001_schemas_and_tables.sql:162-166`:
`source_net_amount numeric(18,2)`, `applied_net_amount numeric(18,2)`,
`amount_delta numeric(18,2)`.
`spec/type-01-card-settlement/inbound/card-settlement-table-definitions.txt`:
`amount_brl decimal(18,2) NOT NULL`.

```sh
grep -n "decimal(18,2)\|numeric(18,2)" \
  legacy/postgres/migrations/001_schemas_and_tables.sql \
  spec/type-01-card-settlement/inbound/card-settlement-table-definitions.txt
```

observed output: multiple `numeric(18,2)` hits in the migration file's
`reporting.card_settlement_reconciliation` columns; one
`decimal(18,2) NOT NULL` hit on `amount_brl` in the table-definitions
dump.

Scope check (money vs. non-money decimals): `contracts/types/05-merchant-fee-assessment/layout.yaml`
carries a `rate_percent` field at `scale: 3` — a percentage rate, not
a money value. Its actual money fields, `gross_amount_brl` and
`assessed_fee_brl`, are both `scale: 2`, consistent with this ADR.
`rate_percent`'s different scale does not contradict this record; it
is explicitly out of scope, not a counterexample.

## Consequences

Pass 3/5 must specify the second plant's amount type as exact decimal
(scale 2) end to end — parsing, comparison, and any downstream
arithmetic; no requirement, task, or eval may accept a float
representation for a money value. A float-based amount would put R-1's
recomputation and R-2's zero-tolerance comparison on an unreliable
footing without ever showing up as a code review complaint.

Re-verify when: a live type's **money** field specifies a scale other
than 2 (Type 05's `rate_percent` is scale 3, but it is a rate, not
money, and is out of this ADR's scope; its money fields remain scale
2).
