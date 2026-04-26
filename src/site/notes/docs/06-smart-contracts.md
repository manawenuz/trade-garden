---
{"dg-publish":true,"permalink":"/docs/06-smart-contracts/","title":"06 — Smart Contract Modules","tags":["trade-protocol","contract","architecture"],"dg-note-properties":{"title":"06 — Smart Contract Modules","tags":["trade-protocol","contract","architecture"],"up":"[[README|Index]]","prev":"[[docs/05-state-machine\|05-state-machine]]","next":"[[docs/07-tokenomics\|07-tokenomics]]"}}
---


# 06 — Smart Contract Modules

Module map and minimal interfaces. Names are illustrative; the goal here is
the *shape* of the system, not Solidity syntax.

## Module dependency graph

```mermaid
flowchart TB
  TRADE[TRADE Token<br/>ERC-20]
  STAKE[StakingVault]
  REG[ActorRegistry]
  ATT[AttestationHub]
  EFAC[EscrowFactory]
  EINST[EscrowInstance<br/>(per trade)]
  POS[PositionNFT<br/>(buyer/seller positions)]
  INS[InsurancePoolFactory]
  IPOOL[InsurancePool]
  DIS[DisputeCourt]
  GOV[Governor]
  TRE[Treasury]
  FEE[FeeRouter]

  TRADE --> STAKE
  STAKE --> REG
  STAKE --> DIS
  STAKE --> GOV
  REG --> ATT
  EFAC --> EINST
  EINST --> POS
  EINST --> ATT
  EINST --> IPOOL
  EINST --> DIS
  EINST --> FEE
  INS --> IPOOL
  IPOOL --> STAKE
  FEE --> TRE
  FEE --> STAKE
  GOV --> EFAC
  GOV --> REG
  GOV --> INS
  GOV --> DIS
  GOV --> TRE
```

## Modules

### TRADE Token (ERC-20)
Capped supply, transferable, used for staking and governance. No rebase, no
fee-on-transfer. See [[docs/07-tokenomics\|07-tokenomics]].

### StakingVault
Holds staked TRADE per role. Each role is a separately-accounted "stream" with
its own slashing authority and unbonding period. Supports:

```text
stake(role, amount)
unbond(role, amount)        // begins cooldown
withdraw(role)              // after cooldown
slash(role, who, amount, beneficiary)   // role-authorised
```

### ActorRegistry
On-chain identity for accredited service providers.

```text
register(role, metadata, kycAttestation)
suspend(actorId, reason)
revoke(actorId)
isAccredited(actorId, role) view returns (bool)
reputation(actorId) view returns (Score)
```

### AttestationHub
Generic signed-statement store. Inspectors, oracles, KYC providers all post
here; consumers (escrow, insurance) read and verify.

```text
post(topic, subjectId, dataHash, signature)
verify(attestationId) view returns (issuer, dataHash, ts)
```

### EscrowFactory & EscrowInstance
Factory deploys minimal-proxy `EscrowInstance` per trade. Each instance
embodies the state machine (see [[docs/05-state-machine\|05-state-machine]]).

```text
EscrowFactory:
  create(terms) returns (escrowId)

EscrowInstance:
  accept()
  deposit()
  nominateInspector(actorId)
  markShipped(bolHash)
  oracleEvent(topic, payload)
  markDelivered(proofHash)
  confirmDelivery()
  dispute(evidenceHash) payable bond
  claimTimeout()
  state() view
  hooks() view
```

### PositionNFT
ERC-721 (or 1155) representing the **buyer position** and optionally the
**seller payout rights**. Transfer is gated by the EscrowInstance to enforce
KYC tier and policy rules. Enables resale and factoring.

### InsurancePool / InsurancePoolFactory
One pool per risk class (e.g. `MARINE_CARGO_A`, `AIR_FREIGHT_B`,
`PERFORMANCE_BOND_TIER_1`). LPs deposit stablecoin; premiums accrue; claims
are paid from pool reserves.

```text
deposit(poolId, amount)
withdraw(poolId, shares)
bindPolicy(escrowId, value, duration) returns (policyId, premium)
fileClaim(policyId, evidenceHash)
ruleClaim(policyId, accept, payout)
```

### DisputeCourt
Selects jurors by stake-weighted draw, manages evidence rounds, tallies votes,
emits ruling, and triggers slashing of incoherent voters.

```text
file(escrowId, evidenceHash, bond)
selectJurors(disputeId)
submitVote(disputeId, commit | reveal)
ruling(disputeId) view returns (Outcome, splitBps)
appeal(disputeId, bond)
```

Detail in [[docs/08-dispute-resolution\|08-dispute-resolution]].

### Governor
Standard token-weighted governor with timelock. Parameters under governance:

- Fee rates (protocol fee, dispute fee, insurance premium spread).
- Stake minima per role.
- Timeouts for state machine.
- Accreditation policy thresholds.
- Treasury allocations.
- Approving new modules (e.g. a new risk-class insurance pool).

### Treasury
Holds protocol fees in stablecoin and TRADE. Spends via Governor.

### FeeRouter
Splits incoming fees between treasury, staker rewards, and burn (if any).

## Upgradeability

- `EscrowInstance`: **non-upgradeable** per trade, by design — both parties
  agreed to a specific code at trade open, immutability is a feature.
- `EscrowFactory`, `Registry`, `InsurancePool`, `DisputeCourt`: upgradeable
  via Governor + timelock. New trades route to new code; existing instances
  finish under old code.
- `TRADE`, `StakingVault`: **non-upgradeable** (highest trust surface).
- A circuit-breaker pause exists on Factory/Pools, callable by the
  bootstrapping admin during phase 1, by Governor thereafter.

## Cross-cutting concerns

| Concern | Handled by |
|---|---|
| Reentrancy | per-instance reentrancy guard; pull-payment for stablecoin out-flows |
| Oracle freshness | per-topic max-staleness; stale events ignored |
| Front-running of dispute filing | commit-reveal on evidence reveal phase |
| MEV on auctioned positions | optional sealed-bid, off-chain order book |
| Gas griefing in juror pool | gas-limit on juror callbacks; jurors paid in TRADE not gas |

---

**See also:** [[docs/02-architecture\|02-architecture]] · [[docs/05-state-machine\|05-state-machine]] · [[docs/07-tokenomics\|07-tokenomics]] · [[docs/08-dispute-resolution\|08-dispute-resolution]]
