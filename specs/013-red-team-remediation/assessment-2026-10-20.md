# AEE Assessment — Spec 013 (after_implement, T40)

**Date:** 2026-10-20 · **Tooling:** AEE engine 1.0.4 (glossa venv pin), threshold 0.70
**Claims assessed:** 47 (`aee-claims.json` v0.2.0) · **Machine record:** `aee-assessment-after-implement.json`
**Outcome: CLARIFY** — "Assessed 47 claim(s); found 223 failure mode(s); 1 claim(s) below the 0.70 confidence threshold."
**Before (after_specify, 2026-10-13):** outcome ITERATE — 47 claims, 222 failure modes, 0 below threshold, every claim a draft.

## What this assessment is

This is the after-implementation re-assessment T40 requires. The claims
bundle was updated to v0.2.0 the way Spec 011's build updated its own:
a claim moved only where shipped, re-runnable evidence says it holds —
the regression tests named in each claim's evidence, plus the
acceptance battery re-run recorded in
`battery-compat-report-2026-10-20.md` (T38/T39). Stage is **checked**
throughout: the evidence is the program's own tests and battery. No
claim is recorded as explained or accepted by this document, and the
engine's scores are reported exactly as the engine produced them.

The battery behind this assessment, in plain terms: all 57 attacks
the 2026-10-13 red team had demonstrated were tried again against the
remediated code. 56 now fail outright. One — presenting a shortened
copy of a ledger as if it were the whole ledger (RTB-11) — still
passes a bare check, because a shortened copy is still a valid
shorter ledger; it fails every pinned check, and the documentation
names the property. No attack succeeded. Every defense that held in
the original engagement was re-run and still holds, and the frozen
and deposited record was re-measured with zero silent changes
(17 of 17 corpus ledgers identical; 26 of 26 deposited file hashes
match).

## Per-claim state (after)

| Claim | Score | State |
|-------|-------|-------|
| CLM-013-01 bundle verdict re-derivation | 1.000 | supported / checked |
| CLM-013-02 anchor binds the presented bundle | 1.000 | supported / checked |
| CLM-013-03 control outcomes ledger-recorded, cross-checked | 1.000 | supported / checked |
| CLM-013-04 predictions inside the freeze commitment | 1.000 | supported / checked |
| CLM-013-05 connector queries ledgered on every path | 1.000 | supported / checked |
| CLM-013-06 freeze wall refuses registration while frozen | 1.000 | supported / checked |
| CLM-013-07 second freeze only by labeled amendment | 1.000 | supported / checked |
| CLM-013-08 freeze mode validated against the enum | 1.000 | supported / checked |
| CLM-013-09 fixture identity committed at registration | 1.000 | supported / checked |
| CLM-013-10 freeze gate refuses non-decisive controls | 1.000 | supported / checked |
| CLM-013-11 re-registration cannot change a statement | 1.000 | supported / checked |
| CLM-013-12 falsifier rule fires on identity-grade matches only | 1.000 | supported / checked |
| CLM-013-13 stance never inverted for negated propositions | 1.000 | supported / checked |
| CLM-013-14 citations normalized once, at intake | 1.000 | supported / checked |
| CLM-013-15 verdict paths require enrolled key + fresh step-up | 1.000 | supported / checked |
| CLM-013-16 admin-plane actions require fresh step-up | 1.000 | supported / checked |
| CLM-013-17 no-MFA sessions cannot make admin mutations | 1.000 | supported / checked |
| CLM-013-18 step-up freshness cannot be replayed | 1.000 | supported / checked |
| CLM-013-19 adjudicator self-grant refused | 1.000 | supported / checked |
| CLM-013-20 revoked key cannot be self-re-enrolled | 1.000 | supported / checked |
| CLM-013-21 small-order keys refused at enrollment | 1.000 | supported / checked |
| CLM-013-22 break-glass scoped to the target tenant | 1.000 | supported / checked |
| CLM-013-23 operational reads require an authenticated identity | 1.000 | supported / checked |
| CLM-013-24 legal hold binds every export-shaped path | 1.000 | supported / checked |
| CLM-013-25 SPRS evidence links must resolve | **0.250** | **draft — below threshold, see below** |
| CLM-013-26 duplicate tenant creation refused | 1.000 | supported / checked |
| CLM-013-27 unknown-tenant failures rate-limited before append | 1.000 | supported / checked |
| CLM-013-28 lane-2 appends are adjudicator acts, recorder named | 1.000 | supported / checked |
| CLM-013-29 faithfulness pins validated; re-pin is adjudicator-only | 1.000 | supported / checked |
| CLM-013-30 deposit approval is two-party in fact | 1.000 | supported / checked |
| CLM-013-31 Phase 6 UI: CSRF token; identity from the session only | 1.000 | supported / checked |
| CLM-013-32 console POSTs carry a per-session CSRF token | 1.000 | supported / checked |
| CLM-013-33 ingest is http/https only; adapter paths confined | 1.000 | supported / checked |
| CLM-013-34 second reviews validate a real, decided, other-decided item | 1.000 | supported / checked |
| CLM-013-35 surface hygiene (errors, empty GRIM, not-issued, disclosure) | 1.000 | supported / checked |
| CLM-013-36 retractions parsed and quarantined | 1.000 | supported / checked |
| CLM-013-37 connector outages raise; only a genuine 404 is not-found | 1.000 | supported / checked |
| CLM-013-38 OpenAlex abstract only under an asserted licence | 1.000 | supported / checked |
| CLM-013-39 Crossref licence entries qualified before use | 1.000 | supported / checked |
| CLM-013-40 sources compared; disagreement quarantines | 1.000 | supported / checked |
| CLM-013-41 enrichment asserts the candidate's identity | 1.000 | supported / checked |
| CLM-013-42 one shared DOI normalizer | 1.000 | supported / checked |
| CLM-013-43 CORE output cap records its reason | 1.000 | supported / checked |
| CLM-013-44 CORE author lists cleaned at ingestion | 1.000 | supported / checked |
| CLM-013-45 connector failures classified once, in the connector layer | 1.000 | supported / checked |
| CLM-013-46 frozen and deposited record verifies identically | 0.999 | supported / checked |
| CLM-013-47 the red-team battery is the acceptance test | 0.999 | supported / checked |

## The one claim below threshold: CLM-013-25, stated plainly

CLM-013-25's text says SPRS input validation "requires each evidence
link to resolve: the referenced target must exist and identify as
claimed." That is not what shipped, and this record does not pretend
otherwise.

Two readings were measured in the battery (probe p12 and supplements
p12c/p12d):

- **Platform-form links** — references to records the platform itself
  keeps (an audit export, a POA&M item, a legal hold, an anchored
  document row) — are resolution-checked at write time. A link naming
  a target that does not exist, or a document anchor naming the wrong
  requirement, is refused. This half of the claim is implemented and
  tested (`tests/test_e7_redteam_platform.py::test_p12_sprs_evidence_links_must_resolve`).
- **Organisation references** — the operator's own evidence held
  outside the platform, which the platform cannot check — are not
  resolved at all. They are recorded and reported as
  **operator-attested**, a separate label that is never presented as
  "resolved." This is deliberate: it is the two-class rule Spec 005's
  own design specifies, recorded in Spec 005's qualification note of
  2026-10-13.

Against the claim's universal wording, the second reading is a
contradiction, and it is entered in the claim's evidence as one
(direction: contradicts). The engine applies its contradiction
penalty (0.750) and scores the claim **0.250, band low** — the single
below-threshold claim in this assessment, and the reason the overall
outcome is CLARIFY rather than a clean pass. The claim stays a draft.
What would close it is a decision on the claim text — narrow the text
to the two-class rule that was designed and shipped, or extend
resolution checking to organisation references — and that decision
belongs to the spec's owner, not to this assessment. Until then the
gap stays on the record, scored, where any reader can see it.

## Cross-reference: the platform trusted-mode residual

One residual from the battery sits near these claims but outside
their texts, and is recorded here so it cannot be lost. Platform
ledgers have no enrolled ledger-signing key of their own, so a
platform ledger cannot yet be verified in trusted mode (against an
expected set of signers) the way a seed-enrolled study store can.
What is true today: a forged platform ledger is refused when opened,
so a forgery cannot be projected, served, or exported. What is
missing is the custody design for such a key — how it is generated,
stored across restarts, and rotated — which is a Spec 005 design
decision, not a defect this spec's build was scoped to fix. The
qualification is written into Spec 005's record:
`specs/005-enterprise-platform/qualification-note-2026-10-20-spec-013-t40.md`.
CLM-013-15 above is scored on its own text (verdict-class append
paths at the service layer), which the battery confirms; it does not
claim the platform-ledger trusted mode this paragraph describes.

## Reading

Forty-six of forty-seven claims stand on shipped tests and a re-run
battery, at the checked stage. The remediation's own acceptance rule
— every demonstrated attack fails, every held defense holds, no
silent change to the frozen record — is met on the evidence in
`battery-compat-report-2026-10-20.md`. The single exception is a
wording gap, not a working attack: CLM-013-25 promises more than the
designed rule does, and the record now says so in the claim's own
evidence, in its score, and here.
