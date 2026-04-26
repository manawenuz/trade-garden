---
{"dg-publish":true,"permalink":"/docs/01-vision/","title":"01 — Vision & Scope","tags":["concept","trade-protocol","vision"],"dg-note-properties":{"title":"01 — Vision & Scope","tags":["concept","trade-protocol","vision"],"up":"[[README|Index]]","next":"[[docs/02-architecture\|02-architecture]]"}}
---


# 01 — Vision & Scope

## The problem

Cross-party commerce has a trust gap. The buyer fears paying for goods that
never arrive or arrive damaged; the seller fears shipping goods that are never
paid for. Today this gap is bridged by:

- **Centralized escrow** (banks, payment processors, marketplaces) — trusted
  but rent-seeking, opaque, and jurisdictionally fragile.
- **Letters of credit** (international trade) — slow, paper-heavy, expensive,
  and gated by banking relationships.
- **Reputation systems** (eBay, Alibaba) — work for low-value, high-frequency
  trade but break down for one-shot, high-value deals.

This project replaces the trusted intermediary with a **protocol** —
deterministic where possible (smart contracts), human where necessary
(inspectors, jurors), and economically aligned through a staked token.

## Three layers of ambition

### Layer 1 — Vanilla escrow (MVP)

Two parties, one good or service, one currency.

1. Seller and buyer agree off-chain, then open an on-chain escrow.
2. Buyer deposits funds into the escrow contract.
3. Seller delivers.
4. Both parties confirm; funds release to seller.
5. Disputes go to a small arbitration committee.

This is a solved problem on-chain (e.g. Kleros, OpenBazaar) — we ship it as the
first milestone to harden the core contracts and the dispute primitive.

### Layer 2 — Real-world goods commerce

Adds the messy parts of physical trade:

- **Inspection** — third-party verifies goods at origin, transit, or destination.
- **Carriage** — freight forwarders, multi-leg shipping, milestone-based release.
- **Insurance** — cargo insurance underwritten on-platform or routed to an external underwriter.
- **Payment terms** — prepayment, partial prepayment, payment on delivery (COD), milestone payments, deferred payment with collateral.
- **Quality assurance** — sampling, testing, documentary checks.

### Layer 3 — Programmable trade instruments

Composable financial primitives over the escrow:

- **Transferable escrow / digital bill of lading** — buyer can sell the
  in-transit good by transferring the escrow position (NFT) to a new buyer
  before delivery.
- **Trade finance** — third parties can lend against an escrow position
  (factoring, invoice discounting).
- **Tokenized insurance pools** — underwriters stake into pools that back specific risk classes.
- **Decentralized governance** — token holders set fees, parameters, accredit inspectors and insurers, and serve as the appellate court of last resort.

> [!warning] Non-goals (explicit)
> - **Custody of physical goods.** The protocol coordinates trust around goods; it never holds them. Inspectors and carriers do.
> - **Replacing carriers, customs, or sovereign law.** We integrate with them.
> - **Stablecoin issuance.** We use existing stablecoins (USDC, etc.) as the settlement currency; the platform token is utility/governance only.
> - **Anonymity.** Counterparties must be identifiable to each other (KYC at the edges); the chain is pseudonymous but legally-enforceable trade requires identity.

## Design principles

1. **On-chain for value, off-chain for data.** The chain holds custody of
   funds, the state machine, and slashable commitments. Bulky data
   (documents, photos, inspection reports) lives on IPFS/Arweave with hashes
   anchored on-chain.
2. **Optimistic by default.** Most trades complete without dispute; the happy
   path must be cheap and fast. Disputes are an expensive exception.
3. **Skin in the game everywhere.** Every actor whose word the system trusts
   (inspector, juror, insurer, governance voter) must stake tokens that can be
   slashed for malfeasance.
4. **Progressive decentralization.** Launch with a multisig "training wheels"
   admin; transfer powers to token-governance as the system matures.
5. **Composable, not monolithic.** Each module (escrow, inspection, insurance,
   dispute) is independently usable. A trade can opt in to inspection without
   opting in to insurance.

---

**See also:** [[docs/02-architecture\|02-architecture]] · [[docs/03-actors-roles\|03-actors-roles]] · [[docs/10-roadmap\|10-roadmap]]
