---
adr: "0000"
status: accepted
date: 2026-08-25
ground: brownfield
converge_pass: 2
spec_ref: ""
supersedes: ""
superseded_by: ""
deciders: "Helena Dias (BRD/tech-spec decider); Structure pass agent (this record)"
---

# 0000 — Context: the ground we stand on

## Terrain

**Brownfield**, with a greenfield artifact inside it. A working legacy
plant already runs end to end — SFTP raw/csv zones, a Java 21 processor,
and a relational store with staging/legacy/reporting schemas — and
already reaches `MATCHED` on the Type `01` steel thread
(`B202607230000001`, net `173.45`). The **second plant** this Structure
pass grounds decisions for does not exist yet (no `modern/` on this
tree). Every ADR below records either (a) a fact already true about the
brownfield legacy plant that the second plant must not contradict, or
(b) a constraint the drop/tech-spec already fixes for the still-unbuilt
second plant.

## Given surface

- Legacy plant, already running: `legacy/publisher/` and
  `legacy/intake/` (raw SFTP zone transitions), `legacy/processor/`
  (Java 21 — parse, validate, sanitize), `legacy/postgres/` (COPY into
  `staging`, apply into `legacy`, refresh `reporting`).
- `contracts/types/01-card-settlement/` — the signed judge for Type
  `01`: `layout.yaml`, `privacy.yaml`, `csv.yaml`, `reconciliation.yaml`,
  and `main/` fixtures. Approved first example: batch
  `B202607230000001`, 2 detail records, source total BRL `173.45`, 0
  rejects.
- `spec/` — the inbound drop: `estate/` (cover, meetings, mail,
  policies) plus one pack per live type, `01`–`05`. Type `06` is not
  present.
- `docs/brd-type-01-card-settlement.md` (Pass 0) and
  `docs/tech-spec-type-01-card-settlement.md` (Pass 1) already exist;
  both carry `verdict: pending` in their Sign-off blocks.
- `evidence/B202607230000001/reconciliation.json` already shows
  `status: MATCHED`, `source_net_amount`/`applied_net_amount:
  "173.45"`, `amount_delta: "0.00"` on this run.

## Build surface

- The second plant itself: no files exist under `modern/` on this tree.
- Its first write (`modern/landing/`, Parquet — ADR 0001) — not built.
- Type `01`'s independent reader, packaged as the five-file unit (ADR
  0002) — not built.
- `docs/seams.md` (Pass 3), `docs/consensus.md` (Pass 4), `docs/tasks/`
  (Pass 5) — not yet written; this pass's ADRs are what they will stand
  on.

## Spec

`docs/tech-spec-type-01-card-settlement.md` — Pass 1 Intent, draft,
sign-off `pending` (owner: Helena Dias, date unset). This grounding
consumes its requirements `R-1`–`R-5`/`W-1`–`W-7` as of this record's
date; it does not consume any requirement this spec does not carry
(e.g. no `R-n` in that file currently states a privacy requirement —
see ADR 0004's Context).
