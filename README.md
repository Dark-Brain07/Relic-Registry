# Relic Registry

An on-chain oracle that classifies material gaps in an identified relic's public provenance timeline without making ownership, authenticity, or legality claims.

## Live Deployment

- Network: GenLayer Studionet (Chain ID `61999`)
- Contract: [`0x6770aE66368Cff2A109E924f6c658F479aa7babA`](https://explorer-studio.genlayer.com/address/0x6770aE66368Cff2A109E924f6c658F479aa7babA)
- Deployer/owner: `0x42d29F098a6fa448B8cCafe50b1291951A9A500d`
- Deploy transaction: [`0xcdb88d51…f23c481`](https://explorer-studio.genlayer.com/tx/0xcdb88d516b0032dec243b2bc7389bd48fb57a281688d2453721e8fd18f23c481)



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
