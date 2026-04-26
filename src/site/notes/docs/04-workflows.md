---
{"dg-publish":true,"permalink":"/docs/04-workflows/","title":"04 — Commerce Workflows","tags":["trade-protocol","workflow","concept"],"dg-note-properties":{"title":"04 — Commerce Workflows","tags":["trade-protocol","workflow","concept"],"up":"[[README|Index]]","prev":"[[03-actors-roles]]","next":"[[docs/05-state-machine\|05-state-machine]]"}}
---


# 04 — Commerce Workflows

This document walks through the canonical trade flows the protocol supports,
each illustrated with a sequence diagram. They are presented in order of
increasing complexity. All build on the core state machine in [[docs/05-state-machine\|05-state-machine]].

## 4.1 — Vanilla escrow (MVP)

Use case: a freelance design job, a P2P used-laptop sale, any low-complexity trade.

```mermaid
sequenceDiagram
  autonumber
  actor B as Buyer
  actor S as Seller
  participant E as Escrow Contract
  participant T as Treasury

  S->>E: createTrade(terms, buyer, amount)
  B->>E: acceptTrade() + deposit(stablecoin)
  E-->>S: notify FUNDED
  S->>S: deliver good / perform service
  S->>E: markDelivered(proof_hash)
  B->>E: confirmDelivery()
  E->>S: release(amount - fee)
  E->>T: fee
  Note over E: state = COMPLETED
```

**Failure paths**: buyer never confirms → seller calls `claimTimeout` after
N days → if no dispute filed, funds release. Either party can `dispute` before
confirmation (see [[docs/08-dispute-resolution\|08-dispute-resolution]]).

## 4.2 — Prepayment with pre-shipment inspection (PSI)

Use case: importing 10,000 units of a manufactured good. Buyer wants
verification before goods leave origin.

```mermaid
sequenceDiagram
  autonumber
  actor B as Buyer
  actor S as Seller
  participant E as Escrow
  actor I as Inspector
  participant R as Registry

  B->>E: createTrade(terms, inspector_required=true)
  S->>E: accept()
  B->>E: deposit(amount)
  B->>E: nominateInspector(I)
  S->>E: agreeInspector(I)
  R-->>E: verify I is accredited & staked
  S->>I: notify ready for inspection
  I->>I: physical inspection at origin
  I->>E: attest(report_hash, PASS|FAIL)
  alt PASS
    E-->>S: release shipment authorization
    S->>S: ship goods
    S->>E: markShipped(BoL_hash)
    Note over B,S: standard delivery flow continues
  else FAIL
    E-->>B: refund(amount - inspector_fee)
    Note over E: state = ABORTED
  end
```

## 4.3 — Milestone payments (multi-stage)

Use case: a long manufacturing run or a multi-leg shipment.

```mermaid
sequenceDiagram
  autonumber
  actor B as Buyer
  actor S as Seller
  participant E as Escrow
  participant O as Oracle/Inspector

  B->>E: createTrade(milestones=[30%, 40%, 30%])
  B->>E: deposit(100%)
  S->>E: claimMilestone(1, evidence)
  O->>E: attest(milestone_1_done)
  E->>S: release(30%)
  S->>E: claimMilestone(2, evidence)
  O->>E: attest(milestone_2_done)
  E->>S: release(40%)
  S->>E: claimMilestone(3, evidence)
  B->>E: confirmFinal()
  E->>S: release(30%)
```

Milestones can be tied to: deposit at factory, BoL issuance, vessel sailing
(AIS oracle), arrival at port, customs clearance, final delivery.

## 4.4 — Buyer-funded escrow with seller bond (quasi-COD)

Use case: buyer refuses to prepay; seller posts a slashable performance bond
to assure they will not extort post-delivery.

```mermaid
sequenceDiagram
  autonumber
  actor B as Buyer
  actor S as Seller
  participant E as Escrow

  S->>E: createTrade(terms, mode=COD, bond=X)
  S->>E: postBond(X TRADE)
  B->>E: accept() + deposit(amount)   deducted from buyer or split per terms
  S->>I: pre-shipment inspection
  I->>E: attest(PASS)
  S->>E: markShipped(BoL_hash, vessel_id)
  SO->>E: vesselDeparted(vessel_id, ts)
  SO->>E: vesselArrived(port, ts)
  alt no incident
    B->>E: confirmDelivery()
    E->>S: release(amount)
    E->>IP: closePolicy(no_claim)
  else incident reported
    B->>E: fileInsuranceClaim(evidence_hash)
    IP->>IP: assess (delegated to surveyor)
    IP-->>E: payout(amount)
    E->>B: payout
    Note over E: subrogation rights to IP
  end
```

## 4.6 — Transferable escrow / in-flight resale

Use case: B1 bought a cargo of crude oil at $80/bbl; while it sails, the spot
price moves to $90 and B1 wants to flip it to B2.

```mermaid
sequenceDiagram
  autonumber
  actor B1 as Buyer 1 (original)
  actor B2 as Buyer 2 (assignee)
  actor S as Seller
  participant E as Escrow (NFT position)
  participant M as Marketplace

  Note over E: Trade is in IN_TRANSIT state.<br/>Buyer position is an ERC-721/1155.
  B1->>M: list(escrowId, askPrice)
  B2->>M: bid / accept
  B2->>E: transferIn(escrowId) + pay(B1)
  E-->>B1: payment from B2 (off-protocol or via M)
  E->>E: buyer = B2
  E-->>S: notify(buyer_changed)
  Note over S: Seller's counterparty risk re-anchored to B2.<br/>KYC of B2 must already be in place.
  S->>E: markDelivered(proof)
  B2->>E: confirmDelivery()
  E->>S: release(amount)
```

Key constraint: seller's interests must be preserved. Transfer is allowed only
if (a) the new buyer passes KYC tier ≥ original, (b) escrow is fully funded
(it is — funds were deposited at trade open), and (c) any insurance policy
follows the position.

## 4.7 — Trade finance / factoring

Use case: seller wants cash now; financier pays seller 95% today, collects
100% from escrow on delivery.

```mermaid
sequenceDiagram
  autonumber
  actor S as Seller
  actor F as Financier
  participant E as Escrow

  Note over E: Trade is funded, shipped.
  S->>F: offer to assign payout rights
  F->>S: pay 95% upfront (off-chain or stablecoin)
  S->>E: assignPayoutRights(F)
  Note over E: payee = F (not S)
  E-->>F: notify
  Note over S,F: Trade continues as before.
  E->>F: release(100%) on confirmation
```

The seller-position can also be tokenised as an NFT, opening the door to
secondary markets and pooled financing — out of scope for the v1 design but
the interfaces should not preclude it.

## Workflow composition

These workflows are not exclusive — a single trade can be:

> *prepaid + PSI-inspected + insured + milestoned + transferable + financed.*

The state machine in [[docs/05-state-machine\|05-state-machine]] is the single canonical flow; each
module is a **hook** that attaches at predefined extension points (`FUNDED`,
`SHIPPING`, `IN_TRANSIT`, `DELIVERED`, `DISPUTED`, `COMPLETED`, `ABORTED`).

---

**See also:** [[docs/05-state-machine\|05-state-machine]] · [[docs/06-smart-contracts\|06-smart-contracts]] · [[docs/09-oracles-inspection-insurance\|09-oracles-inspection-insurance]]
