---
id: T-20260825-type01-writer-landing
title: "Type 01 writer — exact decimal, privacy-clean, deterministic Parquet landing"
status: proposed
leg: swimlane-ingest-landing-leg-04
parent: docs/seams.md#swimlane-1-ingest-landing-type-01
swimlane: ingest-landing
spec_ref: "R-1, R-2, R-3, R-7, R-11"
effort: S
execution_backend: generic
requires:
  network: deny
touches_paths:
  - modern/landing/**
creates_paths:
  - modern/landing/**
budget_iterations: 3
budget_tokens: 0
deciders: "Structure/Tasking pass agent (this record)"
date: 2026-08-25
---

# T-20260825-type01-writer-landing

## Goal

Land Type `01` batch `B202607230000001` (`valid-minimal`) as Parquet
under `modern/landing/`: exact-decimal money, privacy-clean (no raw
PAN/CPF), deterministic across independent runs, and with zero writes
outside `modern/landing/` — frozen folders untouched. This is
`leg-04-writer` from `docs/seams.md`'s Swimlane 1 (Ingest → Landing,
`thread=yes`), the leaf that owns the second plant's published seam.

## Context

Grounds directly in the accepted plan and ADR set — nothing here
re-decides them:

- **Exact decimal** — `docs/adrs/0003-money-is-exact-decimal-never-float.md`
  (accepted): money fields are exact decimal, scale 2, never binary
  float. This leaf's output column for `net_amount_brl` must carry
  that type end to end.
- **Privacy at parse** — `docs/adrs/0004-privacy-dies-at-the-parser-before-landing.md`
  (accepted) and tech-spec `R-11`: PAN and CPF must not survive past
  the parser, "before any Parquet or Gold." This leaf is the last gate
  before landing, so it re-asserts the invariant defensively rather
  than trusting the upstream leg silently: privacy is `leg-03-parser`'s
  job to *apply*, and this leaf's job to *prove* nothing raw leaked
  through by the time it reaches Parquet.
- **Deterministic Parquet under `modern/landing/`** — tech-spec `R-7`
  (first write is `modern/landing/`, not SFTP) and ADR `0001`
  (accepted): the seam Swimlane 2 consumes must be a stable, repeatable
  contract — two independent runs over the same input batch must
  produce byte-identical output, or Swimlane 2's Bronze/Silver/Gold
  layer has no reliable input to conform.
- **No write to frozen folders** — root `README.md` / tech-spec `R-6`:
  `legacy/`, `contracts/`, `gen/`, `infra/` may never be edited to make
  a gate pass, by any pass, ever.
- Reference fixture: `contracts/types/01-card-settlement/main/expected-reconciliation.yaml`
  — batch `B202607230000001`, `source_net_amount`/`applied_net_amount`
  `"173.45"`, `status: MATCHED`, 2 detail records — the authoritative
  judge fixture this leaf's output must reconcile against once built.

**No product code exists yet.** `modern/` is not on this tree. Every
eval below is written to run *now* and is expected to report **RED**
tonight — that is correct, not a defect: "a gate that cannot fail is
worse than no gate" (root `README.md`). Pass 8 (Loop) turns this GREEN
later; this task-spec does not.

## Success Criteria

Four runnable checks. `eval_1`–`eval_3` require the writer to exist and
are expected to fail closed tonight; `eval_4` is a standing invariant
and should already pass.

```bash
BATCH="B202607230000001"
LAND_GLOB="modern/landing/**/${BATCH}*.parquet"

# eval_1 — exact decimal, never float, for the money column.
eval_1_exact_decimal() {
  local f
  f="$(ls -1 $LAND_GLOB 2>/dev/null | head -1)"
  if [ -z "$f" ]; then
    echo "[fail] eval_1_exact_decimal: no landed Parquet found for ${BATCH} under modern/landing/ — writer not yet implemented"
    return 1
  fi
  python3 - "$f" <<'PY' || return 1
import sys
try:
    import pyarrow.parquet as pq
except ImportError:
    print("[fail] eval_1_exact_decimal: pyarrow not available — install it, do not skip this eval")
    sys.exit(1)
schema = pq.read_schema(sys.argv[1])
field = schema.field("net_amount_brl") if "net_amount_brl" in schema.names else None
if field is None:
    print("[fail] eval_1_exact_decimal: no net_amount_brl column in landed Parquet")
    sys.exit(1)
t = str(field.type)
if not t.startswith("decimal"):
    print(f"[fail] eval_1_exact_decimal: net_amount_brl is '{t}', not a decimal type (float/double forbidden)")
    sys.exit(1)
print(f"[pass] eval_1_exact_decimal: net_amount_brl is {t}")
PY
}

# eval_2 — privacy: no raw PAN/CPF; token+last4 / masked forms only.
eval_2_privacy_clean() {
  local f
  f="$(ls -1 $LAND_GLOB 2>/dev/null | head -1)"
  if [ -z "$f" ]; then
    echo "[fail] eval_2_privacy_clean: no landed Parquet found for ${BATCH} under modern/landing/ — writer not yet implemented"
    return 1
  fi
  python3 - "$f" <<'PY' || return 1
import re, sys
try:
    import pyarrow.parquet as pq
except ImportError:
    print("[fail] eval_2_privacy_clean: pyarrow not available — install it, do not skip this eval")
    sys.exit(1)
table = pq.read_table(sys.argv[1])
names = table.column_names
if "pan" not in names and "pan_token" not in names:
    print("[fail] eval_2_privacy_clean: no PAN-shaped column found to check")
    sys.exit(1)
pan_col = "pan_token" if "pan_token" in names else "pan"
cpf_col = "cpf_masked" if "cpf_masked" in names else ("cpf" if "cpf" in names else None)
bad = []
for v in table.column(pan_col).to_pylist():
    s = str(v)
    if re.fullmatch(r"\d{16}", s):
        bad.append(f"raw 16-digit PAN found: {s}")
        break
if cpf_col:
    for v in table.column(cpf_col).to_pylist():
        s = str(v)
        if re.fullmatch(r"\d{11}", s):
            bad.append(f"raw 11-digit CPF found: {s}")
            break
        if s and not re.fullmatch(r"\*{7}\d{4}", s):
            bad.append(f"CPF not in masked seven-star+last4 form: {s}")
            break
if bad:
    print("[fail] eval_2_privacy_clean: " + "; ".join(bad))
    sys.exit(1)
print("[pass] eval_2_privacy_clean: no raw PAN/CPF found; PAN tokenized, CPF masked")
PY
}

# eval_3 — deterministic: two independent runs produce identical output.
# Supply RUN1_PARQUET / RUN2_PARQUET (paths from two separate writer
# invocations over the same batch) once the writer exists.
eval_3_deterministic() {
  if [ -z "${RUN1_PARQUET:-}" ] || [ -z "${RUN2_PARQUET:-}" ]; then
    echo "[fail] eval_3_deterministic: RUN1_PARQUET/RUN2_PARQUET not set — writer not yet implemented; run it twice and set both paths"
    return 1
  fi
  if [ ! -f "$RUN1_PARQUET" ] || [ ! -f "$RUN2_PARQUET" ]; then
    echo "[fail] eval_3_deterministic: one or both run outputs missing"
    return 1
  fi
  local h1 h2
  h1="$(sha256sum "$RUN1_PARQUET" | awk '{print $1}')"
  h2="$(sha256sum "$RUN2_PARQUET" | awk '{print $1}')"
  if [ "$h1" != "$h2" ]; then
    echo "[fail] eval_3_deterministic: run1=$h1 run2=$h2 differ — same input must yield byte-identical output"
    return 1
  fi
  echo "[pass] eval_3_deterministic: run1 and run2 are byte-identical ($h1)"
}

# eval_4 — no writes to frozen folders. Standing invariant; should
# already pass tonight since nothing has touched them.
eval_4_no_frozen_writes() {
  local dirty
  dirty="$(git status --porcelain -- legacy/ contracts/ gen/ infra/ 2>/dev/null)"
  if [ -n "$dirty" ]; then
    echo "[fail] eval_4_no_frozen_writes: uncommitted changes under a frozen folder:"
    echo "$dirty"
    return 1
  fi
  echo "[pass] eval_4_no_frozen_writes: legacy/, contracts/, gen/, infra/ all clean"
}
```

## Boundaries

- `proc.exec`: this task's own evaluations only (`python3`, `sha256sum`,
  `git`, `ls` — all already present in this environment).
- `net.egress`: **deny** (`requires.network: deny`). This leaf reads
  and writes local files only.
- `fs.write`: `modern/landing/**` only, once a future pass implements
  the writer. This task-spec itself writes nothing beyond this file.

## Do-Not-Touch

- `legacy/` — frozen oracle.
- `contracts/` — frozen judge.
- `gen/` — frozen plant.
- `infra/` — frozen plant.
- Anything outside `modern/landing/**` is out of this leaf's write
  scope by default (no `touches_paths`/`creates_paths` entry covers
  it).

## Exit Check

```bash
eval_1_exact_decimal && eval_2_privacy_clean && eval_3_deterministic && eval_4_no_frozen_writes
```
