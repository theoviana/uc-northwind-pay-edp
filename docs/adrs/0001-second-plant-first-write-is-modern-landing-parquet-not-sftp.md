---
adr: "0001"
status: accepted
date: 2026-08-25
ground: brownfield
converge_pass: 2
spec_ref: "R-4, R-7"
supersedes: ""
superseded_by: ""
deciders: "Helena Dias (BRD decider); Structure pass agent (this record)"
---

# 0001 — second plant first write is modern landing parquet, not SFTP

## Context

`docs/tech-spec-type-01-card-settlement.md` (R-4/R-7) already fixes
that the second plant's first write is not SFTP and happens later
(after Consensus) — but deliberately does not pin a destination or
format, to stay above the stack at Intent. Structure must pin the
destination now so Pass 3's seam cut has one fixed landing point to
plan against, rather than each swimlane guessing its own.

## Decision

The second plant's landing target is `modern/landing/`, and its file
format there is Parquet. SFTP — the existing raw/csv transport the
legacy plant owns — is not a valid destination for this write, at any
stage, on either plant.

## Rejected reading

Could have read as: the second plant reuses the legacy plant's
`csv/outgoing/` SFTP zone as a shared handoff point, since both plants
read the same raw bytes. Rejected — the tech-spec's own R-4 states the
first write "is not SFTP," and `docs/README.md` independently fixes the
format and location twice, not just the negative constraint.

## Evidence

`docs/README.md`: "Landing Parquet is product (`modern/`) **after** the
sign — not required Tuesday" and, in "What this folder is not": "Not
`modern/`. First write after the sign is landing Parquet."
`docs/tech-spec-type-01-card-settlement.md` R-4: "the second plant's
first write happens on a later date (Day 2 or after Consensus) and
makes 0 calls to that existing transport layer's publish or claim
path."

```sh
grep -n "landing Parquet" docs/README.md
```

observed output: `docs/README.md:8:9:  Landing Parquet is product` /
`docs/README.md:63: Not \`modern/\`. First write after the sign is
landing Parquet.`

## Consequences

Pass 3 swimlane/seam plans for Type `01` must terminate the second
plant's first physical write at `modern/landing/` in Parquet, never at
`raw/`, `csv/outgoing/`, or any other SFTP zone. Any later pass that
proposes a different first-write destination contradicts this record
and must supersede it explicitly, not silently drift.

Re-verify when: `docs/README.md`'s Pass 8 row, or the tech-spec's R-4,
changes the destination or format.
