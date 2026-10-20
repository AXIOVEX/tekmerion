# Spec 005 — Qualification Note (2026-10-20): platform ledger signing keys (Spec 013 T40)

**What this note is.** Spec 013's acceptance battery
(`specs/013-red-team-remediation/battery-compat-report-2026-10-20.md`)
confirmed the service-layer authority fixes recorded in this package's
qualification note of 2026-10-13 (Spec 013 WS-4). One exposure the red
team demonstrated against the ledger core (unsigned appends) has a
platform half that WS-4 could not close, and this note records it as a
qualification on the claims it touches: **platform ledgers have no
enrolled ledger-signing key of their own.** Nothing in Spec 005's
claims bundle or its earlier assessments is edited by this note; the
affected claims are named below, and the bundle was re-assessed as it
stands (see the re-assessment record at the end).

## The qualification, in plain terms

Every append a *user* makes through the service is signed by a key
enrolled to that user's identity — that rule (CLM-005-03) holds and
was re-verified by the battery. Study ledgers can also be verified in
**trusted mode**: the verifier is given the set of signers it should
expect, and anything unsigned or signed by anyone else fails by name.

Platform ledgers — the platform audit ledger and the deployment
ledger — cannot yet be verified that way, because no signing key is
enrolled *for the platform ledger itself*. The service's own
machinery writes those ledgers, and there is no recorded answer to
the custody questions a platform signing key raises: how the key is
generated, where it is stored so it survives restarts, and how it is
rotated. That is a design decision for this spec's owner. It was
flagged during Spec 013's companion work and deliberately not improvised
by that build.

What is true today, measured by the battery (rt-c p08): a platform
ledger with a forged unsigned append, added through the raw library,
**cannot be reopened** — opening refuses it (`unsigned-event`), so
the forgery can never be projected, served, or exported. The exposure
is therefore mitigated, not open-ended; what is missing is the
stronger, positive check (trusted mode) that study ledgers already
have.

## Affected claims

- **CLM-005-03 (every append is signed by an enrolled key) — qualified
  for the platform ledgers' own writes.** The claim holds for every
  append the service accepts from an identity. It does not yet hold
  in the stronger sense that the platform audit and deployment
  ledgers carry an enrolled ledger-signing key of their own; their
  verification rests on hash-chain integrity and refusal at open, as
  described above.
- **CLM-005-14 (platform-plane actions are recorded as signed,
  hash-chained events in the platform audit ledger) — qualified in
  the same bound.** The events are hash-chained and the actions they
  record carry their actors' signatures and authority checks. The
  ledger as a whole has no enrolled signer, so the standalone
  trusted-mode verification a reader might expect from "signed" is
  not yet available for it.
- **CLM-005-15 (the platform audit record exports for offline
  verification) — qualified narrowly.** Export verification stands as
  measured: bundles verify on their own, and tampering with an export
  fails that verification (the battery re-measured the Spec 005 e6
  audit bundle against its recorded baseline, unchanged). What an
  external auditor cannot yet do is re-verify the exported platform
  ledger in trusted mode against an enrolled platform signer, for
  the same custody reason.

## Other battery observations touching this package

- **CLM-005-12 (residency pins) — a named trust assumption, unchanged.**
  Battery item rt-c p19 (class PLAUSIBLE in the original engagement,
  not a demonstrated attack): a residency-pin move can be satisfied
  by a self-attested anchor receipt — the platform records the
  receipt, and the receipt's truth rests with the anchor target. This
  spec's own text already names that assumption; the battery's
  behavior is unchanged, no Spec 013 claim covers anchor-receipt
  attestation, and the item is recorded here so it stays visible to
  this spec's owner.
- **CLM-005-29 (SPRS-ready control states) — behavior confirmed; the
  open wording item lives in Spec 013.** The battery (p12, p12c,
  p12d) confirms the shipped rule this package's 2026-10-13 note
  describes: platform-form evidence links must resolve or are refused
  at write time; organisation references are labeled operator-attested
  and never presented as resolved. The unresolved item is in Spec
  013's claim CLM-013-25, whose text promises resolution for *every*
  link. In Spec 013's after-implementation assessment
  (`specs/013-red-team-remediation/assessment-2026-10-20.md`)
  CLM-013-25 is scored 0.250 — below threshold — on that contradiction
  and remains a draft pending a decision on its text. CLM-005-29's own
  text makes no such universal promise and is not qualified by this.

## Re-assessment record

The claims bundle (`aee-claims.json` v0.8.0, unchanged) was
re-assessed with the pinned engine (AEE 1.0.4, threshold 0.70) at
`after_implement` on 2026-10-20; the raw record is
`aee-assessment-2026-10-20-qualification.json` in this package, and
the standing `aee-assessment.json` is untouched. The engine's summary,
identical to the standing assessment's (the bundle is unchanged and
the engine is deterministic): **outcome GATHER_EVIDENCE —
"Assessed 31 claim(s); found 132 failure mode(s); 0 claim(s) below
the 0.70 confidence threshold."** The affected claims score 1.0
(band high) on their recorded evidence: CLM-005-03, CLM-005-12,
CLM-005-14, CLM-005-15, CLM-005-29.

That needs one plain sentence of interpretation: the engine scores a
claim from the evidence recorded for it, and these scores say the
recorded evidence supports the claims as the program has evidenced
them. The qualification above is a bound on how CLM-005-03, CLM-005-14,
and CLM-005-15 may be read — it is not expressed in the engine's
numbers, which is exactly why it is written here instead.

## What did not change

The WS-4 authority fixes, the SAML/OIDC/SCIM protocol behavior, the
held hygiene checks, and every verdict and frozen bundle stand as
recorded. Closing this qualification requires a key-custody design
(generation, storage across restarts, rotation) adopted as a Spec 005
decision; until then, trusted-mode verification at the platform layer
remains the one named gap in this package's record.
