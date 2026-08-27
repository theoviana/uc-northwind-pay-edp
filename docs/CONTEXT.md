# CONTEXT — domain glossary

Terms only: canonical name, one-line meaning, the evidence it traces to.
No implementation details, no build notes. Pinned as each term
crystallizes during Structure (Pass 2); see `docs/adrs/` for the
grounding record behind each entry.

- **net amount** — the canonical name for Type `01` trailer bytes
  16–30 (the control total the trailer must equal the sum of the
  detail rows). Ops language calls the same field "settlement total";
  Gold and any later reporting use the layout's name, **net amount**,
  not the ops synonym. **Authoritative evidence**:
  `contracts/types/01-card-settlement/layout.yaml:74` —
  `{ name: net_amount_brl, start: 16, end: 30, ... }` — the signed
  judge's own field name. Corroborating color only (inbound, not
  authoritative): `spec/type-01-card-settlement/inbound/card-settlement-layout-rev3.md`
  ("16–30 | 15 | Net amount overpunch...") vs
  `spec/type-01-card-settlement/inbound/2026-07-02-settlement-total.md`
  (Marina: "Please put settlement total on the recon report"). Resolved
  in favor of the contract's own field name, `net_amount_brl` — a
  mechanical resolution by contract precedence, not a preference call.

- **steel thread** — the single, narrowest end-to-end vertical slice
  built and proven first; tonight and this week, Type `01` card
  settlement is the steel thread, while Types `02`–`05` exist in the
  drop but are not built yet. Evidence: `docs/tech-spec-type-01-card-settlement.md`
  §1; `spec/type-01-card-settlement/README.md`.

- **kept source lie** (source-owned control mismatch) — a batch where
  the source's own declared control total (e.g. a trailer) disagrees
  with the independently recomputed total from its detail rows. The
  declared value is never edited to match the computed one; the batch
  is refused instead. Evidence:
  `spec/type-01-card-settlement/expected/df-source-001.finding.yaml`
  (`declared_net_amount: "173.44"`, `computed_net_amount: "173.45"`);
  ADR `0005`.

- **five-file package** — the per-file-type packaging unit approved
  for the whole modernization effort: model, parser, schema, writer,
  handler — one of each per type, not a monolith. Evidence:
  `spec/estate/meetings/2026-06-09-file-decomposition.md`, D1; ADR
  `0002`.

- **landing** (`modern/landing/`) — the second plant's first write
  location and format (Parquet); it is never the existing SFTP raw/csv
  transport the legacy plant owns. Evidence: `docs/README.md`
  ("Landing Parquet is product... First write after the sign is landing
  Parquet"); `docs/tech-spec-type-01-card-settlement.md` R-4/R-7; ADR
  `0001`.
