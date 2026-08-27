---
adr: "0002"
status: accepted
date: 2026-08-25
ground: brownfield
converge_pass: 2
spec_ref: "R-1"
supersedes: ""
superseded_by: ""
deciders: "Helena Dias (approved D1, 2026-06-09); Structure pass agent (this record)"
---

# 0002 — Type 01 five-file package is the unit

## Context

`docs/tech-spec-type-01-card-settlement.md` R-1 requires "a second,
independent reader" to exist for Type `01`, but does not pin what
files or modules that reader is packaged as. The engagement's own
2026-06-09 file-decomposition sync already settled that shape for the
whole modernization effort, across all five types, before this pass —
Structure grounds it so Pass 3 does not re-decide it per type.

## Decision

Per file type, the modernization plant's unit of packaging is five
files — model, parser, schema, writer, handler. Type `01`'s independent
reader (R-1) uses this same five-file shape as its packaging unit; no
type in this drop gets a different shape.

## Rejected reading

Could have read as: each type ships as one monolithic module (a single
file doing detection, parsing, and writing together), since only Type
`01` is tonight's steel thread and a monolith is faster to stand up
first. Rejected — D1 was approved for "one handler per type, five
files," not for a steel-thread-only exception, and the same sync fixed
it as a rule for all five types, not a Type-`01`-specific shortcut.

## Evidence

`spec/estate/meetings/2026-06-09-file-decomposition.md`, Key Decisions:
"D1 | One handler per type, five files: model, parser, schema, writer,
handler | Helena | Approved."

```sh
grep -n "five files" spec/estate/meetings/2026-06-09-file-decomposition.md
```

observed output: `spec/estate/meetings/2026-06-09-file-decomposition.md:197:|
D1 | One handler per type, five files: model, parser, schema, writer,
handler | Helena | Approved |`

## Consequences

Pass 3 swimlane/seam plans for Type `01` must decompose the
independent reader into exactly these five files/roles, not fewer or
more; Pass 5 task-specs should map close to one leaf per file, not one
leaf per type.

Re-verify when: a later, dated decision record supersedes D1 for this
engagement (no such record exists as of this ADR's date).
