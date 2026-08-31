# Relic Registry

An on-chain oracle that classifies material gaps in an identified relic's public provenance timeline without making ownership, authenticity, or legality claims.

## Live Deployment

- Network: GenLayer Studionet (Chain ID `61999`)
- Contract: [`0x719104cb642088B40A1D30fCc982D283458F2289`](https://explorer-studio.genlayer.com/address/0x719104cb642088B40A1D30fCc982D283458F2289)
- Deployer/owner: `0x42d29F098a6fa448B8cCafe50b1291951A9A500d`
- Deploy transaction: [`0x7b16af4d…f25758`](https://explorer-studio.genlayer.com/tx/0x7b16af4dd822353e92368069dec21e27791c18597135bf81601c016a21f25758)

| Evidence | Transaction | FINALIZED result | Authoritative effect |
|---|---|---|---|
| Registration | [`0x2809fff9…ac9c7b75`](https://explorer-studio.genlayer.com/tx/0x2809fff96ca5b55b14abee7a8d5b0abadb6d530089ea5268149df488ac9c7b75) | SUCCESS, Accepted | Sealed Getty object `92.PB.82`, years 1921–1976, threshold 2, revision 0 |
| First assessment | [`0xb5d2e26c…b0cf6815`](https://explorer-studio.genlayer.com/tx/0xb5d2e26c5f96c30e632b7e540cd9db2b69b2a17c64c29fd3bace6e36b0cf6815) | SUCCESS, MAJORITY_AGREE (3 agree, 1 disagree after rotation) | `ASSESSED`, revision 1, `OPEN_ENDED_GAP` |
| Replay | [`0xbb47852e…249a9861`](https://explorer-studio.genlayer.com/tx/0xbb47852e194b5241a2313094a6f6789efcd5f64035ea1fb5cb221be1249a9861) | SUCCESS, MAJORITY_AGREE (3 agree, 2 idle) | Returned stored result; revision remained 1 |
| Valid revised evidence | [`0x1451a8cc…744dc76e`](https://explorer-studio.genlayer.com/tx/0x1451a8cc94856e6b678288554fc0fc35836af74e56cab8585037fa20744dc76e) | SUCCESS, MAJORITY_AGREE (3 agree, 2 idle) | `REASSESSED`, revision 2; revision 1 remained readable |
| Exact identity mismatch | [`0xe98dd929…47524a8a`](https://explorer-studio.genlayer.com/tx/0xe98dd929c51a75291fa4f3783a7206dc20fd23886c06110175dfcba247524a8a) | SUCCESS, MAJORITY_AGREE (3 agree, 2 disagree) | Returned `UNRESOLVED/IDENTITY_MISMATCH`; state remained revision 2 |
| Non-owner reassessment | [`0xc9ebfb17…b35dd798`](https://explorer-studio.genlayer.com/tx/0xc9ebfb17d13053d4fa916e2f38f354e4b4302ee667d207c062c479feb35dd798) | ERROR rollback, MAJORITY_AGREE (3 agree, 2 idle) | `UNAUTHORIZED: owner only`; state unchanged |
| Stale revision | [`0x0038c2d4…78985f44`](https://explorer-studio.genlayer.com/tx/0x0038c2d4ce313053ea30eae52d63c12185c747a43a1906071c4ceff078985f44) | ERROR rollback, MAJORITY_AGREE (3 agree, 2 idle) | `STALE_REVISION`; state unchanged |

The live object is Getty accession `92.PB.82`, *The Entry of the Animals into Noah's Ark*, evaluated over 1921–1976 with a two-year materiality threshold. Exact transaction inputs are in [`samples/`](samples/).

The exact public read commands, returned values, revision hashes, and transaction-order proof for every state postcondition are in [`docs/verification.md`](docs/verification.md). They can be rerun against the public Studionet contract without a private key.

## Problem and Why GenLayer

Relic provenance is usually prose: custody dates may be approximate, records can conflict, and multiple official sources may describe one chain differently. The contract asks independent validators to reconstruct only the exact object identity and minimum timeline observations, then derives the consequential gap category deterministically.

Do not use GenLayer here when a trusted operator already supplies a complete normalized custody table. A conventional backend is sufficient for deterministic data controlled by one accepted authority.

## How It Works

1. The owner registers institution, accession, title, year window, threshold, and one to three HTTPS URLs under the fixed official roots `metmuseum.org`, `nga.gov`, or `getty.edu`.
2. The leader fetches bounded visible text and asks the model for identity status, timeline condition, and exact gap years only.
3. Validators independently fetch the same sealed evidence and exact-compare every consequential observation.
4. Contract code derives `NO_MATERIAL_GAP`, `BOUNDED_GAP`, `OPEN_ENDED_GAP`, `CONFLICTING_TIMELINE`, or `UNRESOLVED` and its reason code.
5. Successful assessments become immutable revisions; unresolved decisions and rejected transactions cannot corrupt accepted state.

## State Model and Invariants

Lifecycle: `REGISTERED -> ASSESSED -> REASSESSED`.

- Institution, accession number, title hash, object identity hash, target years, and threshold are sealed at registration.
- First assessment may be triggered by anyone; reassessment is owner-only and must name the current revision.
- Replaying the first assessment after revision 1 returns the stored snapshot without another nondeterministic call.
- Bounded years must lie inside the sealed window. The status is derived from duration versus threshold, never accepted directly from a model.
- Open-ended, conflicting, unavailable, malformed, or unresolved evidence stores zero years where applicable.
- Each accepted evidence revision has its own manifest hash and immutable assessment snapshot.

## Public API

Writes:

- `register_object(object_id, institution, accession_number, title, target_start_year, target_end_year, evidence_manifest, materiality_threshold)`
- `assess_provenance(object_id)`
- `reassess_provenance(object_id, prior_revision, evidence_manifest)`

Views:

- `read_gap_status(object_id)` — stable oracle surface returning gap status, years, source count, and reason code.
- `read_object_identity(object_id)` — sealed identity/configuration plus lifecycle and current revision.
- `read_assessment(object_id, revision)` — current or immutable historical assessment.

## Consensus Design and Failure Behavior

Web pages are hostile input, not instructions. URLs are canonicalized, credentials/fragments/non-default ports are rejected, redirects must stay on the same approved root, response size is capped, and markup is stripped before prompting. The prompt fixes identity fields, enum schema, year bounds, and injection resistance. The validator rejects non-return VM outcomes, malformed JSON, unknown reason codes, field divergence, or independently reconstructed observations that differ from the leader.

Evidence unavailable, disallowed redirects, model errors, and unresolved identity return a deterministic `UNRESOLVED` result. Such outcomes preserve the prior accepted lifecycle, revision, manifest, and history. Authorization and stale-revision failures roll back before web/LLM execution.

## Consensus Binding Matrix

| Field | Source | Stored? | Downstream effect | Validator check | Binding mode | Differential test |
|---|---|---:|---|---|---|---|
| `identity_status` | Independent evidence interpretation | Yes | Allows or blocks timeline decision | Exact leader/validator equality and invariants | Exact enum | Matching vs mismatched institution/accession |
| `timeline_condition` | Independent custody reconstruction | Yes via derived status | Selects deterministic gap branch | Exact equality | Exact enum | Continuous vs bounded/open/conflicting |
| `gap_start_year` | Evidence observation | Yes when bounded | Defines interval | Exact equality plus sealed-window bounds | Exact integer | Same condition with diverging start year rejected |
| `gap_end_year` | Evidence observation | Yes when bounded | Defines interval/duration | Exact equality plus ordering/window bounds | Exact integer | Same condition with diverging end year rejected |
| `gap_status` | Contract derivation | Yes | Oracle output | Recomputed from validated observations and threshold | Deterministic | Below/above threshold derive different statuses |
| `reason_code` | Contract derivation or fixed failure class | Yes | Explains machine-readable result | Allowlisted and cross-field checked | Deterministic | Unknown/arbitrary failure reason rejected |
| `source_count` | Canonical manifest | Yes | Evidence-profile signal | Deterministically counted before consensus | Deterministic | One, two, and three-source manifests |
| `manifest_hash` | Canonical manifest | Yes | Revision identity/replay binding | SHA-256 in deterministic code | Deterministic | Same-manifest replay vs revised manifest |

## Tests

```powershell
py -m pip install -r requirements.txt
$env:PYTHONUTF8="1"
genvm-lint check contracts/relic_registry.py
py -m pytest -q -W error::RuntimeWarning
```

Verified result: lint and contract-schema validation pass; all 34 Direct Mode tests pass. Coverage includes registration bounds, official-domain enforcement, deterministic threshold derivation, prompt injection, redirect/web/model failures, replay, immutable revisions, owner/stale checks, and true validator differentials through `run_validator()`.

## Consensus Engineering Lessons

- LLMs should emit observations; contract code should derive consequential statuses and reason codes.
- Cryptographic identity hashes bind state, but plaintext title/institution/accession must still be present for model evaluation.
- Exact-output validators can rotate when narrative evidence supports multiple interpretations; compact official records improve convergence.
- A `FINALIZED` consensus transaction may represent an agreed rollback, so execution result and authoritative readback must both be checked.
- Unresolved evidence is a valid fail-closed result and must not overwrite the last accepted revision.
- Replay should bypass web and LLM work entirely once the sealed assessment exists.

## Reusable Integrations

- A lender can query `read_gap_status` and route `OPEN_ENDED_GAP` or `CONFLICTING_TIMELINE` objects to enhanced review.
- A research DAO can use `read_assessment` to cite the exact immutable evidence revision behind a grant or catalog-cleanup decision.
- A registry can monitor `read_object_identity` for revision changes and refresh downstream risk labels only after reassessment.

## Limitations

This is an evidence-quality oracle, not a title determination, authenticity certificate, sanctions screen, or allegation of unlawful possession. It accepts only the three fixed official domain roots in the deployed revision, does not crawl beyond sealed URLs, and does not interpret images or private dealer records. Public pages can become unavailable, and legitimate ambiguity may produce `UNRESOLVED` or require consensus rotation.

## Repository Structure

```text
contracts/relic_registry.py
tests/test_relic_registry.py
samples/stable-catalog-manifest.json
samples/revised-catalog-manifest.json
samples/identity-mismatch-manifest.json
docs/verification.md
README.md
requirements.txt
LICENSE
.gitignore
```

## License

MIT — see [`LICENSE`](LICENSE).
