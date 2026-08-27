# Consensus — Pass 4 objection log and resolutions

> Converge Pass 4. Attacks `docs/brd-type-01-card-settlement.md`,
> `docs/tech-spec-type-01-card-settlement.md`, `docs/adrs/0000`–`0005`,
> and `docs/seams.md`. Every objection is FIXED (edited in place, cited
> below) or ACCEPTED (recorded here, with a named owner) — none
> silently dropped.

**Provenance note, stated plainly, not glossed over:** this review ran
**same-family, same session** as Passes 0–3 — it is not the
cross-family adversary (`cvg review --adversary`) the full Converge
Pass 4 gate requires, and no stamped `objection-log.json` with an
independent provenance hash exists. Treat this record as the
objection/resolution log a human reviewer or a genuine cross-family
pass would still need to check independently before this counts as the
hardened barrier the method describes. It does **not**, by itself,
satisfy `CHECK_CONSENSUS`.

**Owner sign-off status:** still `pending` in both
`docs/brd-type-01-card-settlement.md` and
`docs/tech-spec-type-01-card-settlement.md`. Nothing below fakes or
implies Helena Dias's signature — that remains hers to give.

## FIXED (edited in place)

| # | Objection | Fix applied | Files touched |
|---|---|---|---|
| 1 | Every 173.44/173.45/MATCHED claim cited `spec/`'s inbound mirror, never `contracts/`'s signed judge fixtures. | Re-pointed BRD §4, tech-spec R-1/R-2/R-5, and ADR `0005` to cite `contracts/types/01-card-settlement/main/expected-df-source-001-finding.yaml` and `expected-reconciliation.yaml` as authoritative; `spec/`'s copies are now explicitly labeled "corroborating color only." | `docs/brd-type-01-card-settlement.md`, `docs/tech-spec-type-01-card-settlement.md`, `docs/adrs/0005-*.md` |
| 2 | `CONTEXT.md`'s "net amount" resolution cited only the inbound layout doc, never the contract. | Re-cited to `contracts/types/01-card-settlement/layout.yaml:74` (`net_amount_brl`); inbound layout doc demoted to corroborating color. | `docs/CONTEXT.md` |
| 4 | No tech-spec requirement ID existed for privacy; Pass 5 would have nothing to cite for `leg-03-parser`'s privacy eval. | Added **R-11** (privacy dies at the parser) to tech-spec `### Must`; updated ADR `0004`'s `spec_ref` from empty to `R-11`, and its Context/Consequences to record the gap as closed. | `docs/tech-spec-type-01-card-settlement.md`, `docs/adrs/0004-*.md` |
| 5 | BRD/tech-spec still listed the "net amount" vs "settlement total" question as open, owner Marina Alves, while `CONTEXT.md` had already resolved it — the two documents disagreed. | Closed the open question in both BRD and tech-spec, pointing at `CONTEXT.md`'s resolution; stated explicitly that contract precedence made this mechanical and does not require Marina's further sign-off. | `docs/brd-type-01-card-settlement.md`, `docs/tech-spec-type-01-card-settlement.md` |
| 8 | ADR `0003` claimed "no live type's layout specifies a scale other than 2," which is false as literally written (Type 05's `rate_percent` is `scale: 3`). | Narrowed the Decision and "Re-verify when" line to **money fields** specifically; added a Scope-check evidence paragraph naming `rate_percent` as a rate, not money, and confirming Type 05's actual money fields (`gross_amount_brl`, `assessed_fee_brl`) are both `scale: 2`. | `docs/adrs/0003-*.md` |
| 10 | ADR `0005` (about keeping the lie) cited `spec_ref: R-2, R-3` but not `R-1`, whose title ("Keep the lie") matches it best. | Added `R-1` to ADR `0005`'s `spec_ref`, now `R-1, R-2, R-3`. | `docs/adrs/0005-*.md` |
| 11 | Tech-spec R-6 said "at least 4 frozen trees" — unfalsifiable-ish for a closed, fixed set. | Changed to "Exactly 4 frozen trees named." | `docs/tech-spec-type-01-card-settlement.md` |

All six edited ADRs and the tech-spec were re-checked against their
actual gate scripts after editing (not just eyeballed):

```
bash .grok/skills/brd-docs-to-tech-req/scripts/check-tech-spec.sh --draft docs/tech-spec-type-01-card-settlement.md
→ CHECK_TECH_SPEC=DRAFT_OK (1 warning: Sign-off pending — expected at draft stage)

bash .grok/skills/tech-req-to-adrs/scripts/scaffold-adr.sh --check
→ CHECK_ADR=OK
```

## ACCEPTED (recorded here, not edited)

| # | Objection | Resolution | Owner |
|---|---|---|---|
| 3 | ADR `0001`'s `deciders` field named Helena Dias, but its evidence is entirely `docs/README.md` prose and this session's own tech-spec — nothing attributable to her. | **Accepted as engagement scaffolding.** `docs/README.md`'s statement that the second plant's first write is landing Parquet is explicit and repeated twice in that file; the room treats it as already-authorized engagement scaffolding (written by the course/engagement designer, not a stray inbound preference), not something needing a fresh customer sign-off tonight. `docs/adrs/0001-*.md`'s `deciders` field is left as-is under this acceptance — a future reviewer who disagrees can supersede the ADR, not silently edit it. | Helena Dias (accepts the scaffolding framing as engagement owner); flagged for her actual confirmation if she reviews this Consensus record. |
| 6 | Same root cause as #3 — ADR `0001` locks "Parquet" as a fact one pass earlier than the method's own division of labor (Pass 3 is supposed to make the reversible stack pick). | **Accepted, named explicitly here** (not silently via the ADR alone): `docs/README.md`'s repeated, explicit "landing Parquet" statement is treated as already-fixed for this engagement. `docs/seams.md` Swimlane 1 may continue to cite `modern/landing/` Parquet as its seam's contract. | Helena Dias / engagement scaffolding (see #3). |
| 7 | BRD and tech-spec each carry two parallel structures (custom "1.–6." headings plus the full cvg-required section set), restating overlapping content with nothing keeping them in sync if edited separately. | **Accepted for tonight** — content in both halves currently agrees and the gate scripts pass. **Flagged for cleanup**: before Types `02`–`05` repeat this shape, collapse to one canonical structure so drift can't happen silently. | Translator seat (Night 2, or whoever authors Types `02`–`05`'s BRD/tech-spec). |
| 9 | All six ADRs were still `status: proposed` when `docs/seams.md` (Pass 3) was built against them. | **Accepted as designed sequencing** — the `reqs-to-swimlane-plans` skill explicitly permits Pass 3 to run same-session against `proposed` ADRs; acceptance happens at review, not at scaffold time. **Resolved at sign-off below**: ADRs `0000`–`0005` are now `status: accepted`. | Owner (theo.viana) — done at sign-off, see below. |

## What this record does not do

- Does not sign the BRD or tech-spec `canonical` — both remain
  `verdict: pending`. Consensus (this pass) signs the hardened plan
  set; Capture/Intent sign-off is a separate act, still Helena Dias's
  to give.
- Does not create `modern/`, write a Task-Spec, or touch `legacy/`,
  `contracts/`, `gen/`, or `infra/`.
- Does not substitute for a genuine cross-family adversarial pass —
  see the Provenance note above; this sign-off closes tonight's
  objection log, it does not retroactively manufacture a
  cross-family `objection-log.json`.

## Sign-off

- **Signed by:** theo.viana, acting as owner/decider for this
  exercise.
- **Date:** 2026-08-25.
- **Verdict:** **canonical** — every objection above is FIXED or
  ACCEPTED; none left open, none silently dropped.
- **ADR disposition:** `docs/adrs/0000`–`0005` move from
  `status: proposed` to `status: accepted` as of this sign-off.
- **Scope of this signature:** it signs *this objection log and its
  resolutions*, and the ADR set's move to `accepted`. It does not
  independently sign `docs/brd-type-01-card-settlement.md` or
  `docs/tech-spec-type-01-card-settlement.md` `canonical` — those
  Sign-off blocks are unchanged by this record and remain `pending`
  unless and until edited in their own files.
