# Public Studionet Verification

This document makes the E2E state claims reproducible from the public contract. View calls do not require a private key and do not mutate state.

## Target

- Network: GenLayer Studionet, Chain ID `61999`
- Contract: [`0x719104cb642088B40A1D30fCc982D283458F2289`](https://explorer-studio.genlayer.com/address/0x719104cb642088B40A1D30fCc982D283458F2289)

Select Studionet once:

```powershell
genlayer config set network=studionet
```

## Current Identity and Lifecycle

```powershell
genlayer call 0x719104cb642088B40A1D30fCc982D283458F2289 read_object_identity --args [YOUR_OBJECT_ID]
```

*(Once you execute test transactions, you can record the verified returns here).*

## Current Oracle View

```powershell
genlayer call 0x719104cb642088B40A1D30fCc982D283458F2289 read_gap_status --args [YOUR_OBJECT_ID]
```

*(Once you execute test transactions, you can record the verified returns here).*

## Immutable Revision 1

```powershell
genlayer call 0x719104cb642088B40A1D30fCc982D283458F2289 read_assessment --args [YOUR_OBJECT_ID] 1
```

*(Once you execute test transactions, you can record the verified returns here).*

## Scenario-to-State Proof

| Scenario | Public transaction evidence | Reproducible state proof |
|---|---|---|
| Deploy | [`0x7b16af4d…f25758`](https://explorer-studio.genlayer.com/tx/0x7b16af4dd822353e92368069dec21e27791c18597135bf81601c016a21f25758) | Contract address resolves and all three public views execute. |
