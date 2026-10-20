# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-10-20

These entries record the red-team remediation (Spec 013). An adversarial
review of the first release found ways to make the record say things
that were not true; each entry below names what was wrong in plain
terms and what changed.

### Added

- Retraction checking for cited works. Citation records now carry a retraction state read from Crossref and OpenAlex, and a retracted work is quarantined: it contributes no evidence until an adjudicator decides otherwise. Before, a retracted paper counted the same as any other source.
- Lane-2 records now name who recorded them. Digestion and acceptance records (the human-paced half of a verdict) can only be made by an adjudicator, their references must point at records that exist, and the record shows the recorder beside the named human.
- Second reviews are checked against the record: the item must exist and be decided, the reviewer must not be the person who decided it, and agreement must be a real yes/no, not text that gets quietly converted.

### Changed

- Bundle verification now re-derives every verdict. A study bundle's verdicts are recomputed from the bundle's own evidence and compared with the issued verdicts, and anchor checks bind the bundle actually presented — so editing a bundle's evidence or verdicts after the fact, even with the manifest rehashed, fails verification instead of passing.
- Citation errors are reported honestly. When Crossref or OpenAlex is rate-limited, down, or out of budget, the connectors now raise the failure and the pipeline records an error. Before, those cases were recorded as "not found" — the same answer as a citation that does not exist — which quietly corrupted the counts kill criteria rest on. A genuine 404 is still "not found".
- Sources are compared before any is believed. Resolution consults all configured connectors and compares what they return; a disagreement is recorded and the item quarantined, instead of the first answer winning silently. Licence rules are enforced per source: an abstract is served only when a licence covers it, and a text-mining licence no longer unlocks content.
- Verification results report signature state. Study and bundle verification now surfaces how many signatures were checked and failed, which events are unsigned, and which actors a trusted key map does not cover — the full result shape of the Axiomere verification policy this release is built against, including expected-head pins that catch a shortened copy of a ledger.

### Fixed

- Freeze integrity. A claim can no longer be registered against a frozen study, and a study cannot be frozen a second time to launder what was registered after the first freeze; a second freeze is refused and only an explicit, labeled amendment act can follow. Freeze modes are validated, predictions are inside the freeze commitment, and fixture identities are committed at registration so a fixture cannot be swapped after freezing.
- Exporter honesty for older studies. Re-exporting a study whose frozen registration committed no fixture material no longer stamps a fresh "invalid, zero verdicts issued" replay over the recorded history: controls whose fixtures were not presented are reported as not executed, and the recorded verdicts stand as issued.
- Falsifier and stance checks no longer misfire. The falsifier-naming rule fires only on identity-grade matches (an exact DOI, or a surname and year in citation position with a second identity element), not on substrings another work happens to contain; negatively worded propositions no longer get their stance inverted; DOIs are normalized once at intake so every form of the same DOI behaves the same.

### Security

- Platform authority holds at the service layer, not just at sign-in. Verdict-class appends require an enrolled key and a fresh step-up on every path — an API key alone cannot issue a verdict. Admin-plane actions (role grants, identity-provider configuration, key issuance, break-glass policy) require MFA evidence where policy requires it and a fresh, server-issued step-up that cannot be replayed. Nobody can grant themselves the adjudicator role, a revoked key cannot be re-enrolled without a tenant-admin countersign, and weak (small-order) signing keys are refused at enrollment.
- Break-glass and reads are scoped. Break-glass authority is evaluated in the target tenant only and acting requires the invoker's own session; operational reads require an authenticated identity; legal holds bind every export-shaped path, including the break-glass audit export.
- Web surfaces take identity from the session, never from the request. The Phase 6 UI and the console verify a per-session CSRF token on state-changing POSTs, POST identity comes from headers and cookies only (a query string cannot name an actor), and identity cookies are `SameSite=Strict`. Ingestion accepts only http/https URLs, and adapter file paths are confined to the workspace. Error messages no longer leak filesystem paths.

## [0.1.0] - 2026-10-13

Initial public release. Published to PyPI on 2026-10-13; the git tag `v0.1.0` marks the exact commit the published files were built from (e545ad5).

### Added

- Claim registration where every claim must carry a falsifier — the test that would show it is wrong — before it can be registered.
- Citation checking through Crossref, OpenAlex, and CORE, with the licence for each record recorded alongside it.
- Study freezing, adjudication, and pre-registered evaluation checks, so verdict rules and predictions are fixed before results are seen.
- Signed, append-only audit bundles that can be verified on their own, without the engine.
- Deposit of study records to OSF and Zenodo.
- An MCP server with 29 tools, plus an agent skill, so other programs can run studies end to end.
- Domain adapters — crystallography validation (checkCIF/PLATON scope), reproducibility certification (CODECHECK), and statistical-consistency checking (GRIM/statcheck) — each with a registered statement of what its check cannot establish.
- The enterprise platform: sign-in by OIDC and SAML, SCIM provisioning, role-based access, multi-tenancy, break-glass emergency access that requires a second authorizer, and compliance evidence including the CMMC responsibility matrix and the POA&M register.
- Public worked studies: the Phase 7 evaluation and the mathematics-claims study, both frozen before they ran and deposited with their records.

### Fixed

- The evaluation caught a verifier defect that compared only verdict counts, which let a rehashed verdict-flip pass as unchanged. Fixed with full payload comparison and a regression test, before the frozen evaluation run.
