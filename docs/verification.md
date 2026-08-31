# Public Studionet Verification

This document makes the E2E state claims reproducible from the public contract. View calls do not require a private key and do not mutate state.

## Target

- Network: GenLayer Studionet, Chain ID `61999`
- Contract: [`0x7927BcDf4788e7a60E95d0bCaed8Bd4D1c05D4Bd`](https://explorer-studio.genlayer.com/address/0x7927BcDf4788e7a60E95d0bCaed8Bd4D1c05D4Bd)
- Object ID: `mpgr-getty-001`
- Registration: [`0x2809fff96ca5b55b14abee7a8d5b0abadb6d530089ea5268149df488ac9c7b75`](https://explorer-studio.genlayer.com/tx/0x2809fff96ca5b55b14abee7a8d5b0abadb6d530089ea5268149df488ac9c7b75)

Select Studionet once:

```powershell
genlayer config set network=studionet
```

## Current Identity and Lifecycle

```powershell
genlayer call 0x7927BcDf4788e7a60E95d0bCaed8Bd4D1c05D4Bd read_object_identity --args mpgr-getty-001
```

Verified return:

```text
[
  'J. Paul Getty Museum',
  '92.PB.82',
  '2d9ca9c3b7c5e263c821a821d766cd2bce704b987e1e6f2ebf0b61c3f0e2bc4a',
  '08311d9055532e62a9365e951e620a58a6065e38516e92401686c96952b10d55',
  '54c04c35971b6cd61563b3302c85f59dadc2d7959782ee521c60204a3792c8d3',
  1921,
  1976,
  2,
  'REASSESSED',
  2
]
```

The third and fourth hashes bind the exact title and full object identity. The fifth is the current revision-2 manifest hash.

## Current Oracle View

```powershell
genlayer call 0x7927BcDf4788e7a60E95d0bCaed8Bd4D1c05D4Bd read_gap_status --args mpgr-getty-001
```

Verified return:

```text
[ 'OPEN_ENDED_GAP', 0, 0, 2, 'OPEN_ENDED_GAP_DETECTED' ]
```

## Immutable Revision 1

```powershell
genlayer call 0x7927BcDf4788e7a60E95d0bCaed8Bd4D1c05D4Bd read_assessment --args mpgr-getty-001 1
```

Verified return:

```text
[
  'IDENTITY_MATCH',
  'OPEN_ENDED_GAP',
  0,
  0,
  3,
  'OPEN_ENDED_GAP_DETECTED',
  'f78d80c0683fdc3d7c55046039a08259d59672d1339612d579d086971e2bd915',
  1
]
```

This manifest hash is the canonical hash of [`samples/stable-catalog-manifest.json`](../samples/stable-catalog-manifest.json). It is still readable after revision 2.

## Immutable Revision 2

```powershell
genlayer call 0x7927BcDf4788e7a60E95d0bCaed8Bd4D1c05D4Bd read_assessment --args mpgr-getty-001 2
```

Verified return:

```text
[
  'IDENTITY_MATCH',
  'OPEN_ENDED_GAP',
  0,
  0,
  2,
  'OPEN_ENDED_GAP_DETECTED',
  '54c04c35971b6cd61563b3302c85f59dadc2d7959782ee521c60204a3792c8d3',
  2
]
```

This manifest hash is the canonical hash of [`samples/revised-catalog-manifest.json`](../samples/revised-catalog-manifest.json).

## Scenario-to-State Proof

Explorer timestamps and the transaction list establish the order below; the view calls establish the final authoritative storage and both immutable history entries.

| Scenario | Public transaction evidence | Reproducible state proof |
|---|---|---|
| Deploy | [`0x1b3843e9…fd639198`](https://explorer-studio.genlayer.com/tx/0x1b3843e9c000caa169a881dc971a2719e9abe95ab79580ef5d0f13d9fd639198) | Contract address resolves and all three public views execute. |
| Register revision 0 | [`0x2809fff9…ac9c7b75`](https://explorer-studio.genlayer.com/tx/0x2809fff96ca5b55b14abee7a8d5b0abadb6d530089ea5268149df488ac9c7b75) | Receipt contains the exact Getty input; current identity retains its sealed title/identity hashes, years, and threshold. |
| First assessment to revision 1 | [`0xb5d2e26c…b0cf6815`](https://explorer-studio.genlayer.com/tx/0xb5d2e26c5f96c30e632b7e540cd9db2b69b2a17c64c29fd3bace6e36b0cf6815) | Receipt returns `OPEN_ENDED_GAP`; `read_assessment(..., 1)` returns the accepted immutable snapshot. |
| Replay remains revision 1 | [`0xbb47852e…249a9861`](https://explorer-studio.genlayer.com/tx/0xbb47852e194b5241a2313094a6f6789efcd5f64035ea1fb5cb221be1249a9861) | Replay receipt has no nondeterministic output and returns revision-1 data. The later successful reassessment names `prior_revision=1`, proving replay did not increment it. |
| Stale revision fails unchanged | [`0x0038c2d4…78985f44`](https://explorer-studio.genlayer.com/tx/0x0038c2d4ce313053ea30eae52d63c12185c747a43a1906071c4ceff078985f44) | Receipt rolls back `STALE_REVISION`; the later successful reassessment still requires and accepts `prior_revision=1`. |
| Valid reassessment creates revision 2 | [`0x1451a8cc…744dc76e`](https://explorer-studio.genlayer.com/tx/0x1451a8cc94856e6b678288554fc0fc35836af74e56cab8585037fa20744dc76e) | Current identity is `REASSESSED/2`; both `read_assessment(..., 1)` and `read_assessment(..., 2)` return distinct manifest-bound snapshots. |
| Non-owner fails unchanged | [`0xc9ebfb17…b35dd798`](https://explorer-studio.genlayer.com/tx/0xc9ebfb17d13053d4fa916e2f38f354e4b4302ee667d207c062c479feb35dd798) | Sent after revision 2 by `0x5B6465…9b545a`; receipt rolls back `UNAUTHORIZED: owner only`; current identity/history remain revision 2. |
| Identity mismatch fails closed | [`0xe98dd929…47524a8a`](https://explorer-studio.genlayer.com/tx/0xe98dd929c51a75291fa4f3783a7206dc20fd23886c06110175dfcba247524a8a) | Sent last with `prior_revision=2`; receipt returns `UNRESOLVED/IDENTITY_MISMATCH`. Current identity, current manifest hash, and both history reads remain the accepted revision-2 state. |

The contract address page publicly lists all transactions in order. Any judge can rerun the four commands above to verify the current state and immutable revision history against those receipts.
