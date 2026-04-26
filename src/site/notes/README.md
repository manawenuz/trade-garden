---
{"dg-publish":true,"permalink":"/readme/","title":"Trade Protocol — Index","tags":["moc","trade-protocol"],"dg-note-properties":{"title":"Trade Protocol — Index","aliases":["MOC","Home","Index"],"tags":["moc","trade-protocol"]}}
---


# Decentralized Trade Escrow Platform

A hybrid on-chain / off-chain escrow protocol for real-world commerce — from
simple peer-to-peer goods and services to international import/export with
inspections, insurance, financing, and in-flight resale.

> [!info] Status
> This vault currently contains the **design specification only**.
> Implementation follows once the design is locked.

## Document map

| #   | Document                                               | Purpose                                                                  |
| --- | ------------------------------------------------------ | ------------------------------------------------------------------------ |
| 01  | [[docs/01-vision\|Vision & Scope]]                          | Problem, MVP, extended scope                                             |
| 02  | [[docs/02-architecture\|System Architecture]]               | On-chain / off-chain split, components                                   |
| 03  | [[docs/03-actors-roles\|Actors & Roles]]                    | Buyer, seller, inspector, carrier, insurer, arbitrators, token holders   |
| 04  | [[docs/04-workflows\|Commerce Workflows]]                   | Prepayment, COD, inspection, multi-leg shipping, in-flight resale        |
| 05  | [[docs/05-state-machine\|Escrow State Machine]]             | Canonical states & transitions                                           |
| 06  | [[docs/06-smart-contracts\|Smart Contract Modules]]         | Module breakdown & interfaces                                            |
| 07  | [[docs/07-tokenomics\|Tokenomics & Staking]]                | Utility token, staking, slashing, fee flow                               |
| 08  | [[docs/08-dispute-resolution\|Dispute Resolution]]          | Juror selection, evidence, appeals                                       |
| 09  | [[docs/09-oracles-inspection-insurance\|Oracles & Bridges]] | Bridging physical world to chain                                         |
| 10  | [[docs/10-roadmap\|Roadmap]]                                | Phased rollout                                                           |

> [!info] Auto-generated index (Obsidian + Dataview)
> The table above is the source of truth on GitHub. Inside Obsidian with the
> Dataview plugin enabled, the block below renders the same list directly from
> the vault frontmatter — useful for catching drift.

```dataview
TABLE WITHOUT ID
  file.link AS "Document",
  title AS "Title"
FROM #trade-protocol
SORT file.name ASC
```

## Reading order

If you are new, read in numerical order. If you only want the gist:
[[docs/01-vision\|01-vision]] → [[docs/02-architecture\|02-architecture]] → [[docs/04-workflows\|04-workflows]] → [[docs/07-tokenomics\|07-tokenomics]].

## Viewing the docs

The docs render fine on GitHub (mermaid diagrams included), but they are
authored as an **[Obsidian](https://obsidian.md) vault** — `[[wikilinks]]`,
YAML frontmatter, and `> [!note]` callouts. For the best experience:

1. Clone the repo:
   ```bash
   git clone https://github.com/manawenuz/decentralized-trade-escrow.git
   ```
2. In Obsidian, choose **Open folder as vault** and pick the cloned directory.
3. Recommended core plugins: *Graph view*, *Backlinks*, *Outgoing links*, *Tags pane*.
4. Recommended community plugins: *Breadcrumbs* (consumes the `up` / `prev` /
   `next` frontmatter for navigation), *Dataview* (auto-build doc indexes
   from frontmatter).

On GitHub, wikilinks display as literal `[[text]]` rather than as links —
that is expected. Everything else (headings, tables, mermaid) renders normally.

## Tag map

- `#trade-protocol` — vault-wide tag, applied to every doc in this design set
- `#concept` — protocol concepts and primitives
- `#architecture` — system topology, on/off-chain split
- `#vision` — scope, goals, non-goals
- `#workflow` — end-to-end commerce flows
- `#contract` — on-chain modules and interfaces
- `#actor` — roles and responsibilities
- `#tokenomics` — token, staking, fees
- `#governance` — disputes, voting, parameters
- `#roadmap` — phasing and milestones
