# Spec 013 — Battery re-run and backward-compatibility report

Date: 2026-10-20. Tasks: T38 (battery re-run, CLM-013-47) and T39
(backward compatibility, CLM-013-46). This is the final acceptance
evidence for the red-team remediation: every demonstrated attack from
the 2026-10-13 red team was re-run against the remediated code, every
held defense was re-checked, and the frozen and deposited record was
re-measured for silent changes.

## Code under test

- Tekmerion `f472ccb4e81d80dcbb343630fa4ac1ed811456c6` (main after
  PRs #34–#43)
- Axiomere core `cc452998c7bdbdd3fae50964733fef5f9bed0d3e`
- axiomere-python `09d4bd704cf6acbe31521ea94943b1712abfb450`
  (core pinned to cc45299)

Full test suites, run in environments built the way CI builds them:

- Tekmerion: **570 passed, 3 skipped** (the expected state).
- Axiomere core: `cargo test -p axiomere` **32 passed, 0 failed**;
  `cargo check -p axiomere-py` clean.
- axiomere-python: **17 passed**.

## Method

Probes were copied out of `redteam-2026-10-13/` into a battery
working tree and run against fresh clones of the three repositories
at the commits above. The originals were not modified (one exception,
disclosed at the end of this report). Where a probe no longer ran
as-is because an API changed, the probe was adapted as little as
possible to attempt the same attack through the current surface, and
the adaptation is noted next to its result. A probe that could not be
run is reported as not run, with the reason — never counted as a
pass.

## T38 — the five critical attacks

All five critical attacks from the red team now fail.

1. **Checkpoint-root transplant (RTB-01).** A ledger whose checkpoint
   events were re-signed wholesale by an attacker's key is refused:
   `verify_file` returns not-ok at sequence 5 with reason
   `checkpoint-root-mismatch` (the checkpoint root is recomputed, not
   read from the file), and opening the database refuses. The fully
   re-signed transplant built with the red team's own resign tool
   (rebuilt against the current core) fails the same way.
2. **Bundle evidence not re-derived (RTA-01).** Bundle verification
   now re-derives every standing verdict from the bundle's own
   evidence. The red team's edited bundle (a verdict rewritten from
   contradicted to supported, ledger untouched) is refused with the
   reason stated in plain terms: the standing verdict "does not
   re-derive from the bundle's evidence."
3. **Post-freeze claim injection and second-freeze laundering
   (RTA-03, RTD-01).** Registering a claim against a frozen study is
   refused ("registration is frozen; no claim may be registered
   against a frozen study"); a second freeze is refused and points at
   the amendment act instead. The same refusals hold on the MCP
   surface.
4. **Unsigned appends (RTB-02/03/04, rt-c p08).** In trusted mode,
   signature-stripped and attacker-signed events fail verification
   with named reasons (`unsigned-event`, `untrusted-actor`,
   `signer-key-mismatch`). At the platform layer, a ledger with a
   forged unsigned append appended through the raw library cannot be
   reopened: opening refuses with `unsigned-event` at sequence 2, so
   the forgery can never be projected, served, or exported. One
   residual stays open by design decision, stated in §Residuals.
5. **Retractions ignored (RTE-01).** A retracted work (the Wakefield
   record, fed from the captured live payloads) now carries
   `retraction: "retracted"` on the retrieved record and is
   **quarantined** at pipeline level: quarantined items contribute no
   evidence until adjudicated.

## T38 — RT-A (verdict machinery): 9 demonstrated attacks, all fail

| Probe | Attack | Outcome |
|---|---|---|
| RTA-01 | Edited bundle verdict | Fails — verdict re-derivation (above) |
| RTA-02 | Anchor checked against the wrong bundle | Fails — `verify_anchor` binds the presented bundle's own hash; the tampered bundle fails, and the pristine Phase 7 v2 bundle checked against the *v1* Zenodo record now fails with a precise reason (see T39 — this pairing change is designed) |
| RTA-03 | Post-freeze injection; second freeze | Fails — both refused (above) |
| RTA-04 | Fixture swap after freeze | Fails — run stays invalid; export refused: the presented fixture "does not match the fixture identity committed in the frozen registration (CLM-013-09)" |
| RTA-05 | Vacuous freeze (non-decisive control) | Fails at the freeze gate — freeze check returns four named failures. Note, stated precisely: calling the evaluator directly, bypassing the freeze gate, still evaluates vacuously; the evaluator was never the gate — the certified path (freeze) refuses |
| RTA-06 | DOI-substring / surname-year falsifier collisions | Fails — neither fires; control case correct |
| RTA-07 | Stance negation dropped | Fails — negated propositions now contradict, affirmed support |
| RTA-08 | Bundle document edits (probe p09) | The original probe self-skipped because RTA-04's export refusal removed its input bundle. Adapted probe p09b (an honest bundle made with current code, the same six edits) — all six edits fail with named reasons (control outcomes not ledger-recorded, CLM-013-03; fixture identity, CLM-013-09) |
| RTA-09 | DOI forms (bare / URL / uppercase) | Fails — all forms resolve identically |

Held defenses H-1 to H-8 (verdict swap, falsifier edit, margin edit,
ledger payload edit, ghost citation, metadata-only fold, dependency
cycles, empty falsifier) all still hold, each caught with a named
failure.

## T38 — RT-B (ledger core): 11 demonstrated attacks

Ten fail outright; one is contained by the shipped design change
(RTB-11, below).

| Probe | Attack | Outcome |
|---|---|---|
| RTB-01 | Checkpoint transplant | Fails — `checkpoint-root-mismatch` (above) |
| RTB-02 | Signature stripping | Fails — trusted mode refuses all three strip variants (`unsigned-event`); bare mode names the unsigned events in the result instead of hiding them |
| RTB-03 | Unsigned append | Fails — verification rejects (`unsigned-event`, events 7–8 named) |
| RTB-04 | Attacker-signed append | Fails — `untrusted-actor`, actor named |
| RTB-06 | Snapshot forgery | Fails — `snapshot-id-mismatch` at sequence 7; current view refuses |
| RTB-07 | Out-of-range / non-finite numbers | Fails — append itself crashes the write with an exact-range error; 2^70, −2^65, NaN, ±inf each refused ("non-finite float cannot be stored"); the largest exact integer round-trips |
| RTB-08 | Non-canonical bytes | Fails — blank-line, CRLF, extra-field, and space variants all `non-canonical-line`; a mangled re-serialization also fails |
| RTB-09 | Duplicate payload / event | Fails — `non-canonical-line` / `unparseable-line` |
| RTB-11 | Truncated prefix presented as the whole | Contained — see below |
| RTB-12 | Dead-record attacks | Fails — resurrection refused in the core ("record r1 is dead; death is terminal"); a dead snapshot without a reason is refused; status "Dead" fails verification (`invalid-status`); a raw forged dead snapshot fails `snapshot-id-mismatch` |
| RTB-13 | Malformed files raise instead of returning a verdict | Fixed — `verify_file` returns a full result with ok=false and a named reason for malformed files |

RTB-05 (trusted mode not exposed to Python callers) is closed: the
Python `verify`/`verify_file` and the CLI both take a trusted key map
and an expected head, and this battery used them throughout. RTB-10
(the format document left three readings open) is closed by the
current document text, which pins raw UTF-8 strings, signed
never-zero-padded exponents, and prev_hash as hex text; a clean-room
implementation of the current text verifies both honest corpus
ledgers, and the document's own worked example reproduces its printed
hash exactly (`cbaa00f5…fea`).

**RTB-11, stated plainly.** A truncated prefix of a ledger is still,
by itself, a valid shorter ledger — that cannot be changed without
breaking what a ledger is, and bare verification still says so. What
the remediation shipped is the pin: verification against an expected
head. Measured here: the truncated prefix fails pinned verification
(`shortfall` against a full pin, `root-mismatch` against a root-only
pin), and the intact victim ledger passes the same pin. The API
documentation states that a bare pass on a truncated prefix is
expected and that the pin is required — the property is named, not
silent.

Held defenses: the surgery family (delete, duplicate, edit, replay,
splice, swap in the middle of a ledger) is caught on every variant
with a named break (`seq-gap`, `hash-mismatch`); an attacker signing
as the victim fails in trusted mode (`signer-key-mismatch`); score
forcing still stores 0.0; bad magic and unparseable files fail
closed; a missing native extension raises an ImportError with build
instructions; the two-writer case fails verification and refuses to
open. Cross-surface agreement (the p15 differential, adapted to the
three current surfaces — the original shipped-vs-fresh premise no
longer exists): **40 of 40 corpus ledgers, zero disagreements**
between the axiomere-python bindings, the bindings bundled in the
Tekmerion environment, and the Rust CLI — same verdict, event count,
break position, and reason on every file.

## T38 — RT-C (platform): 14 demonstrated attacks, all fail

| Probe | Attack | Outcome |
|---|---|---|
| p01 | Admin mutations without MFA step-up | Fails — grant, IdP configuration, API key, and break-glass policy all refused |
| p02 | Admin-plane mutations from sessions that never stepped up | Fails — all four refused: "admin-plane mutations require a fresh step-up at the service layer (CLM-013-16)". Adapted probe p02b: the original probe's world now builds its admin sessions through the step-up ceremony, so the adaptation uses fresh never-stepped sessions for the same four actions |
| p03 | Step-up replay / forged freshness | Fails — caller-asserted nonces refused, a consumed token replayed against its own or a fresh challenge refused, verdict attempts after replay refused, a stale token refused. Adapted probe p03b runs the real ceremony as its control (the original probe's construction no longer exists) |
| p04 | API key used to sign a verdict | Fails — verdict refused; out-of-class scope, out-of-scope, and revoked-key refusals all still hold |
| p05 | Self-grant of the adjudicator role | Fails — refused by name (CLM-013-19). The probe's later legitimate-chain setup still uses the old caller-asserted step-up and stops there; the refusal it hits is the new ceremony requirement, and the legitimate path is covered by the repository suite |
| p06 | Revoked key re-enrolled by its owner | Fails — self re-enrollment refused for every role (CLM-013-20: re-enrollment requires a tenant-admin countersign); the key stays revoked and appends stay refused. The probe's adjudicator half was adapted to the server-issued challenge ceremony |
| p07 | Small-order (weak) key enrollment with forged proof | Fails — forged proof-of-possession enrollment refused; appends with the forged signature refused |
| p08 | Unsigned appends to a platform ledger | Fails — see critical 4 |
| p09 | Break-glass invoked across tenants | Fails — cross-tenant invocation refused; the role-less and unauthorized paths refused |
| p10 | Operational reads with no credential | Fails — the reads now require a session; adapted probe p10b: no token, a bogus token, and another tenant's token are all refused on all four reads, and the same-tenant control works |
| p11 | Break-glass audit export walks around a legal hold | Fails — export under hold refused on the tenant path and the break-glass path; erasure and pin moves under hold refused. Adapted probe p11b passes a session to the hold read-back (which now requires one) |
| p12 | SPRS score shaped by operator assertion | Fails in its demonstrated form — assertion prose is refused at input; platform-form evidence links that do not resolve are refused (supplements p12c/p12d: fake `export:`/`poam:`/`hold:` ids refused, a document anchor naming the wrong requirement refused, a real export id accepted and reported "resolved"). Organization references the platform cannot check are recorded and reported as **operator-attested**, never silently equivalent — see the claim-text note in §Residuals |
| p16 | Re-creating a tenant to overwrite its pin and marking | Fails — duplicate creation refused ("tenant ids are unique for the life of the deployment"); the existing tenant record is unchanged |
| p17 | Flooding the deployment audit ledger via unauthenticated SCIM calls for unknown tenants | Bounded — the first 10 calls append failure records, the next 20 are rate-limited (the original run appended all 30 with no limit). The probe's own result text is hardcoded from the original run; its measured counts show the limit working |

Held protocol batteries, re-run in full: **SAML** (baseline passes;
signature, binding, replay, wrapping, and encryption attacks all
refused), **OIDC** (baseline and first login pass; all attacks
refused), **SCIM** (cross-tenant, admin-role mapping, and rename
collision all refused) — the same results as the original battery,
including the SAML IdP-initiated (unsolicited) acceptance the
original recorded. Hygiene probe p18 (adapted to the challenge
ceremony): delegated-custody seeds never reach a ledger, API keys are
stored hashed, and the webhook receiver takes the first delivery and
refuses replays and forgeries — all held.

## T38 — RT-D (MCP, console, UI): 9 demonstrated attacks, all fail

| Probe | Attack | Outcome |
|---|---|---|
| RTD-01 | Freeze wall bypass (post-freeze registration) | Fails — new and existing claim registration both refused by the freeze wall on the MCP surface |
| RTD-02 | Re-registering a claim with a changed statement and falsifier | Fails — "statement is immutable after registration; create a superseding claim", before and after citation attachment |
| RTD-03 | Lane-2 spoofing; faithfulness re-pin by a researcher | Fails — digestion/acceptance refused by the role gate and the lane-2 state stays "unexplained"; re-pin refused ("replacing it requires the adjudicator role", CLM-013-29) |
| RTD-05 | Freeze-mode attacks | Fails — a bogus mode is refused by name (CLM-013-08); the exploratory path issues no verdict |
| RTD-06 | Deposit self-approval | Fails — no approver, a blank approver, and self-approval all refused, the last by name (CLM-013-30); an unverifiable bundle is also refused |
| RTD-07 | UI drive-by (cross-site POST with query-string identity) | Fails — adapted probe p14b serves a copy of the frozen math-study workspace under the current UI: the three drive-by POSTs (run, export, deposit; foreign Origin, no cookie, no token) execute nothing and the study ledger gains **0 events** (the original wrote 3) |
| RTD-08 | SSRF / local-file ingest | Fails — `file:` and other non-http schemes refused by name (CLM-013-31/32 class); no local file content ingested |
| RTD-09 | Fuzzing, second-review and reference checks | Fails — empty GRIM runs refused (CLM-013-35); a second review by the decider or on an unknown item refused (CLM-013-34); acceptance references must resolve to a record in the study; decisions are appends (re-deciding refused) |
| RTD-10 | Console duplicate-tenant and CSRF | Fails — a duplicate create appends no tenant event and changes nothing; a foreign-Origin POST with a session cookie creates nothing; plane separation, logout, and login-fixation checks all held. The console's served build identity is `git-f472ccb4e81d+src-1ca99fede658` — the battery commit (T39's BUILD_ID assertion) |

Held defenses: the **role matrix** (29 tools × 4 roles) still holds —
the only changes are in the fix's direction (researchers additionally
refused the two lane-2 tools, which is RTD-03's fix) and one refusal
precedence change (an empty GRIM run is refused for emptiness before
the role gate; with real cases the role refusal fires, verified
separately). Environment-identity checks are unchanged (missing or
blank actor and non-listed roles stop the server with the same
messages). The falsifier wall refuses empty, whitespace, null,
missing, and non-string falsifiers with the original Article II
wording (probe p03, re-run as p03b with claim ids in the grammar the
current server enforces, so each case reaches the check it was
written for; its other dispositions match the original exactly).
Error responses redact workspace paths.


## T38 — RT-E (connectors and supply chain): 14 demonstrated items

Connector probes were re-run offline against stubbed transports
(code probes) and live against the real Crossref, OpenAlex, and CORE
APIs (live probes). One adaptation: the current connectors raise on
transport errors, where the original code returned normally, so the
probe's error cases were wrapped to record the raise — the same
pattern the probe already used for CORE.

| Item | Finding | Outcome |
|---|---|---|
| RTE-01 | Retractions ignored | Fails — quarantine (above) |
| RTE-02 | 429/500/budget exhaustion fabricated as "not found" | Fails — all raise named errors; the pipeline records outcome `error`; a genuine 404 still records `not_found` |
| RTE-03 | OpenAlex abstract served with no licence asserted | Fails — the abstract is withheld without a licence, like the other sources |
| RTE-04 | First connector wins silently | Fails — sources are compared; the disagreement is recorded and quarantines (the probe's fabricated year disagreement now surfaces as a stage-2 disagreement instead of passing silently) |
| RTE-05 | A text-mining licence entry unlocks the abstract | Fails — the TDM entry is recorded as the licence basis but the abstract is not kept and bundle redistribution is not permitted |
| RTE-06 | Enrichment applied to a different work | Fails — a record whose id is not the cited DOI is passed over |
| RTE-07 | DOI forms behave differently per connector | Fails — bare, `doi:`-prefixed, uppercase, trailing-punctuation, and URL forms all normalize to the same request for every connector |
| RTE-08 | CORE output cap degrades silently | Answered — the cap still stops at 3 outputs, but the item now records the reason `output-cap-reached`, distinguishable from "no licensed deposit exists" |
| RTE-09 | CORE author lists polluted | Fails — the live Schuch record now yields its six real authors, clean (no duplicates, no malformed names) |
| RTE-10 | Error classification differs by route; queries logged after execution | Addressed — classification is uniform in the connector layer (the resolve-route raises above), and query ledgering ahead of execution is part of the verdict-machinery checks in RT-A |
| RTE-11 | Wheel builds embed the core at floating main | Fixed and verified — the release workflow checks the core out at the pinned commit `cc45299…` with an explicit never-float note |
| RTE-12 | Wheel smoke test passes on length-correct garbage | Fixed and verified — the smoke test runs a full signed round trip and asserts every signature checked in trusted mode |
| RTE-14 | No SHA pinning; the publish step floats on a branch ref | Fixed and verified on the publish path — every action in the axiomere-python release workflow, including the PyPI publish action, is pinned to a full commit SHA. Tekmerion's CI-only workflow (no publish step, no token) still uses floating major-version tags; noted as a hygiene residual |
| RTE-18 | Tracked `.pyc` files in the Tekmerion repo | Fixed and verified — 0 tracked `.pyc` files |

## Residuals and items outside the demonstrated set (stated, not buried)

- **Platform ledger signing keys (Critical 4, platform half).** The
  measured behavior: a forged platform ledger cannot be reopened,
  projected, or exported — verification at open refuses it. What the
  platform does not yet have is an enrolled ledger-signing key of its
  own, so platform ledgers cannot yet be verified in *trusted* mode
  the way seed-enrolled study stores are. That is a key-custody
  design decision (generation, storage, rotation) belonging to
  Spec 005, flagged during the companion work; it is not closed by
  this battery and must not be read as closed.
- **RT-C p19 (PLAUSIBLE class, not a demonstrated finding).** A
  residency-pin move can still be satisfied by a self-attested
  anchor receipt: the platform records receipts, and the receipt's
  truth is the anchor target's — a trust assumption the Spec 005
  text itself names. The probe's behavior is unchanged. No Spec 013
  claim covers platform anchor-receipt attestation; it is recorded
  here for the owner, outside T38's demonstrated tally.
- **CLM-013-25 wording vs implementation.** The claim text says
  every evidence link must resolve. The implementation enforces
  resolution for platform-form links (measured, p12c/p12d) and
  labels organization references — which the platform cannot check —
  as operator-attested. The T40 assessment should reconcile the
  claim text with the implemented rule.
- **AK-1 / AK-2 (RT-B acknowledged items).** Recomputing a genesis
  block remains possible as self-consistency (a wholly forged ledger
  still verifies *bare*); the anchored mitigation is what changed,
  and it is measured working (RTB-01). Wholesale erasure (AK-2)
  remains documented-unimplemented and was not re-probed.
- **Not run, with reasons.** (1) RT-C p05's legitimate verdict chain
  past its attack half — the probe's own setup uses the retired
  caller-asserted step-up; the attack half is demonstrated refused
  and the legitimate path is covered by the repository suite.
  (2) RTA-10 — PLAUSIBLE class, no probe existed; the machinery
  (prediction hashing into the registration commitment, query
  ledgering and cross-checking) was confirmed in code and is covered
  by the suite. (3) The RT-E deposits probe scripts — they compare
  deposited bundles against the record; that comparison is
  re-measured directly in T39 below (hashes and anchor checks), and
  the deposits themselves were verified at deposit time by the
  records work. (4) RTE-29/30/31 are governance items held for the
  owner, not code outcomes.

## T39 — Backward compatibility (CLM-013-46)

Measured Spec 011-style: the same frozen-set driver, the same
baselines recorded before the remediation, re-run against the code
above. **Zero silent changes.** Every item below matches its recorded
baseline exactly, except one designed change, named with its cause.

- **Frozen set — 9 of 9 lines identical** to the recorded
  post-remediation baseline (`frozen-set-postfix.txt`), line for
  line: Phase 5 bundles v1–v4 verify ok; math bundles v1–v2 verify
  ok with the identical verdict sets (CLM-M-110 contested, 210
  contested, 310 inconclusive, 410 inconclusive, 510 supported);
  Phase 7 bundles v1–v2 verify ok, run outcome valid, identical
  verdict sets (CLM-E-001 supported, 002 supported, 003
  inconclusive, 004 contradicted, 005 inconclusive); the Spec 005 e6
  platform-audit bundle still returns not-ok with the same format
  failures — that not-ok *is* its baseline. (The comparison
  normalizes only the repository path prefix embedded in the e6
  bundle's file-not-found strings.)
- **Axiomere compat corpus — 17 of 17 ledgers identical**: bare
  verification ok with the recorded event counts (93/95/95/45/11/14/
  18/20/20/8/41/45/45/11/18/93/41), and every ledger opens. This set
  includes the ledgers of the deposited bundles: Phase 5
  (10.5281/zenodo.23311479, 23317023), the math study
  (10.5281/zenodo.23317259), and Phase 7 (10.5281/zenodo.23319126).
- **Phase 5 demo bundle v4 as deposited** (10.5281/zenodo.23355310)
  is the frozen-set v4 line above: verifies ok, unchanged.
- **In-repo frozen artifacts — 4 of 4 verify ok**
  (`frozen-artifacts/`: Phase 5 v1, v3, v4 and math v1).
- **Deposited spec claim bundles — 26 of 26 anchored files match.**
  Each spec's `anchors.json` records the SHA-256 of its deposited
  claim-bundle files; every in-repo file hashes to its anchored
  value (Specs 001, 002, 003, 004, 006, 007, 008, 009, 010, 011,
  012 — the eleven deposits made by the records work).
- **OSF-anchored Phase 7 bundle — pairing verified live.**
  `verify_osf_anchor` for bundle v2 against its OSF registration
  (84dk7, project dvrew) returns ok with no failures: registration
  accepted, anchor values stated, all five components and their
  files present.
- **The one designed change, named.** Phase 7 bundle v2 checked
  against the Zenodo record 10.5281/zenodo.23319126 now returns
  not-ok — because that record anchors *v1's* bundle, and
  CLM-013-02 requires an anchor check to bind the presented bundle's
  own hash. The bundle itself verifies ok; its true anchor is the
  OSF registration above, which passes. This is RTA-02's fix working
  as designed, not a compatibility break: no verdict, no ledger, and
  no file changed — only a mismatched pairing that used to pass now
  correctly fails, with the reason stated.

## Disclosure

One battery probe (`rt-e/probes/connectors/code_probes.py`) writes
its results file to a path hardcoded inside the original red-team
tree. Running the battery copy overwrote
`redteam-2026-10-13/rt-e/outputs/connectors/code_probe_results.json`
with the battery run's results, and the live-probe run refreshed the
captured raw API payloads in the same tree. The original results
remain on record in the team's FINDINGS.md and summary files and in
the program synthesis; the battery's own outputs are preserved with
this work. No other original evidence was modified; all other probes
ran from copies against copies or fresh scratch state.

## Verdicts

- **T38: met.** Every in-scope DEMONSTRATED attack was re-run:
  57 demonstrated attacks across the five teams — 56 fail outright,
  and the remaining one (RTB-11) is contained by the shipped head-pin
  with the bare-ledger property explicitly documented. Every held
  defense that was re-run still holds. The residuals above are
  named, scoped, and none of them is a demonstrated attack still
  succeeding silently.
- **T39: met.** The full frozen and deposited set was re-measured:
  9/9 frozen-set lines identical, 17/17 corpus ledgers identical,
  4/4 in-repo frozen bundles ok, 26/26 anchored files
  hash-identical, the OSF pairing verified live — and the single
  changed outcome is the designed CLM-013-02 anchor-pairing change,
  named with its cause.
