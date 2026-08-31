# Public Studionet Verification

This document makes the E2E state claims reproducible from the public contract. View calls do not require a private key and do not mutate state.

## Target

- Network: GenLayer Studionet, Chain ID `61999`
- Contract: [`0x6770aE66368Cff2A109E924f6c658F479aa7babA`](https://explorer-studio.genlayer.com/address/0x6770aE66368Cff2A109E924f6c658F479aa7babA)

Select Studionet once:

```powershell
genlayer config set network=studionet
```

## Current Identity and Lifecycle

```powershell
genlayer call 0x6770aE66368Cff2A109E924f6c658F479aa7babA read_object_identity --args [YOUR_OBJECT_ID]
```

*(Once you execute test transactions, you can record the verified returns here).*

## Current Oracle View

```powershell
genlayer call 0x6770aE66368Cff2A109E924f6c658F479aa7babA read_gap_status --args [YOUR_OBJECT_ID]
```

*(Once you execute test transactions, you can record the verified returns here).*

## Immutable Revision 1

```powershell
genlayer call 0x6770aE66368Cff2A109E924f6c658F479aa7babA read_assessment --args [YOUR_OBJECT_ID] 1
```

*(Once you execute test transactions, you can record the verified returns here).*

## Scenario-to-State Proof

| Scenario | Public transaction evidence | Reproducible state proof |
|---|---|---|
| Deploy | [`0xcdb88d51…f23c481`](https://explorer-studio.genlayer.com/tx/0xcdb88d516b0032dec243b2bc7389bd48fb57a281688d2453721e8fd18f23c481) | Contract address resolves and all three public views execute. |
