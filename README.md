<!-- solve-problem:8f659cf6-9566-4f49-b0ce-c483f7331541 -->
# Cross-domain order-auction settlement

A [Tangle Solve problem](https://solve.tangle.tools/problems/8f659cf6-9566-4f49-b0ce-c483f7331541).

## What is the problem?

**Formal setting.** An *order-flow auction* (OFA) is a mechanism in which a seller of a transaction (a user or wallet, possibly with private ordering preferences, e.g., "execute only if sandwiching does not occur") submits an order or intent; searchers/Builders bid for the right to include and settle it, and the auction determines the winner and the payment to the user. Classically these auctions run within one domain (one blockchain with one consensus/finality rule).

Here, the order's settlement is *cross-domain*: the auction's outcome must be executed on two or more chains with **heterogeneous finality** — different confirmation latencies, reorg probabilities, and finality semantics (e.g., probabilistic fast-finality chains vs. delayed hard-finality chains).

**The core question.** Design an OFA/settlement mechanism such that:

1. **No free abandonment option.** A winning bidder cannot profitably walk away from its bid when conditions on one domain turn adverse (e.g., its own chain reorgs, a price moves, a contingent leg fails) without paying for that option. Formally: the mechanism must be *incentive-compatible under asynchronous, partially-reversible execution*, i.e., bidding truthfully and completing settlement must be an equilibrium, or deviation/abandonment must be priced (escrow, penalties, option premiums).
2. **Quantified privacy/capital frontier.** Characterize the tradeoff between (a) how much information about the order and preferences is revealed to counterparties and the public, and (b) the capital locked in escrow/bonds to deter abandonment, as a function of the finality-gap between domains.

**Ambiguity note.** The census record is informal; the above is the most reasonable formalization. No canonical formal statement exists — part of the problem is producing one. A game-theoretic model with explicit timing, information sets, and strategy spaces (bid, execute, abandon) must be fixed before proofs are possible.

## Why does it matter?

Cross-domain intents (e.g., cross-chain swaps, arbitrage between L1s and L2s, bridged settlement) are already a large fraction of MEV extraction, and today's solvers effectively hold unpriced American-style abandonment options — they bid aggressively, then drop orders when a leg becomes unprofitable, externalizing the cost to users as failed transactions and slippage. A mechanism with priced abandonment would eliminate these hidden settlement subsidies, protect users with private ordering preferences (anti-sandwiching, fairness constraints), and give protocol designers a principled escrow-sizing rule. It also directly answers an open question flagged in the 2023 cross-domain MEV systematization.

## What's already known?

- **Single-domain OFAs are reasonably well understood**: sealed-bid and progressive auctions for order flow, builder-mediated inclusion (MEV-Boost), and privacy-preserving matching with ordering preferences (orderflow auctions in the Flash Boys 2.0 lineage). These all assume one domain with (effectively) atomic inclusion, so abandonment is not an issue.
- **Cross-domain MEV is systematized but not solved**: the SoK (McMenamin 2023, arXiv:2308.04159) catalogs cross-domain MEV vectors, notes that cross-domain execution lacks atomicity, and identifies cross-domain/settlement-related auction design as an open problem — it maps the terrain rather than constructing mechanisms.
- **Related building blocks**: asynchronous market clearing theory (batch auctions with latency), contingent-claims/option pricing for settlement risk, escrow-based solver mechanisms in intent protocols (e.g., solver bonding in cross-chain intent systems like Across/UniswapX-style RFQ — practitioner designs, largely unproven), and commit-reveal/HTLC-style conditional settlement. None of these jointly solve truthfulness + abandonment pricing + privacy under heterogeneous finality with proofs.
- **Why they fall short**: single-domain auction theory gives no handle on partial reversibility; HTLCs handle conditional execution but not strategic bidding; practitioner escrow designs are heuristic with no incentive-compatibility analysis and no privacy accounting.

(Verify practitioner details — much of the intent-protocol mechanism literature is blog/documentation-grade.)

## What would count as solving it?

Any of the following, in increasing strength:

1. **A formal model** with explicit domains $(C_1,\dots,C_n)$, finality times/reorg distributions, user preferences, bidder strategy space {bid value, execute, abandon}, and utility functions — plus a precise definition of "free abandonment option" and its price.
2. **An impossibility or lower bound**: e.g., no mechanism with escrow below $X$ (as a function of the finality gap $\Delta$ and adversarial reorg probability) can be incentive-compatible; or a privacy–capital tradeoff lower bound (any mechanism hiding preference type $\theta$ requires escrow $\geq f(\Delta,\theta)$).
3. **A mechanism with proofs**: an escrow/penalty schedule and bid rules such that (i) truthful bidding + completion is a Bayesian (or dominant-strategy) equilibrium, (ii) abandonment is exactly priced (the option has zero net subsidy), (iii) a quantified privacy guarantee (e.g., differential-privacy-style bound on what bidders learn about the order), with matching or near-matching bounds.
4. **A construction + simulation** demonstrating the tradeoff curve empirically on realistic finality parameters (Ethereum L1 vs. L2 rollups) — weaker than proofs but acceptable for the applied variant.

## How to start

1. **Read the SoK first** (arXiv:2308.04159), especially the taxonomy of cross-domain MEV vectors and the section on mitigations/settlement — it tells you exactly which failure modes any mechanism must neutralize.
2. **Build the minimal two-domain model**: one fast chain with reorg probability $p$, one slow final chain, one order needing settlement on both, one risk-neutral bidder. Compute the value of the abandonment option under naive escrow-free bidding. This calculation *is* the formalization of "hidden subsidy."
3. **Import the right tools**: treat the abandonment option as an American option and ask what escrow equals its price; connect to mechanism design with verifiable states (penalty/commitment mechanisms) and to asynchronous market-clearing results for the timing structure.
4. **Main obstacle**: on-chain verification of "who abandoned whom" is itself cross-domain — proving the fast-chain reorg or failed leg to the escrow domain is expensive or impossible. Consider relaying assumptions, optimistic verification, or designing so penalties only require local evidence.

## Key references

- McMenamin, P. (2023). *SoK: Cross-Domain MEV*. arXiv:2308.04159. Systematizes cross-domain MEV across chains with different consensus/finality, catalogs attack vectors including settlement-related ones, and poses auction/settlement design under heterogeneous finality as open. https://arxiv.org/abs/2308.04159
- Daian et al. (2019). *Flash Boys 2.0: Frontrunning, Transaction Reordering, and Consensus Instability in Decentralized Exchanges*. arXiv:1904.05234. Foundational empirical/theoretical treatment of ordering auctions on a single domain; baseline for what OFA designs assume about atomicity. https://arxiv.org/abs/1904.05234
- Park, A. et al. — order-flow-auction literature (e.g., *A2MM / Inside the Cross-Domain MEV*, and practitioner OFA designs such as MEV-Share (Flashbots)). Provides single-domain OFA and privacy-preserving flow mechanisms whose assumptions break cross-domain. (verify exact citations — this area spans papers and protocol documentation)
- Ferreira et al. (2021). *Commit-Reveal Mechanisms for AMMs* and related verifiable-outcome mechanism-design work; useful for penalty/commitment technique under adversarial chains. (verify)

## Warnings

- **The problem has no canonical formal statement.** Any paper must first fix a model; referees will attack timing and information-set assumptions aggressively. Justify each assumption (reorg distributions, relay latency, bidder risk-neutrality).
- Don't re-derive single-domain OFA results — the difficulty is *entirely* in partial reversibility and asynchronous legs.
- Watch for implicit trusted relayer/oracle assumptions; a mechanism whose penalty enforcement requires an honest cross-domain observer may be vacuous.
- Escrow "solutions" that simply over-collateralize are trivially safe but dead capital — the interesting content is the *quantified* frontier, not existence.
- The census record's phrase "contingent-claim settlement" hints the intended framing is options-pricing-based; keep that lens.

---
*`BCGT-000303` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **A formal model** with explicit domains $(C_1,\dots,C_n)$, finality times/reorg distributions, user preferences, bidder strategy space {bid value, execute, abandon}, and utility functions — plus a precise definition of "free abandonment option" and its price.
2. **An impossibility or lower bound**: e.g., no mechanism with escrow below $X$ (as a function of the finality gap $\Delta$ and adversarial reorg probability) can be incentive-compatible; or a privacy–capital tradeoff lower bound (any mechanism hiding preference type $\theta$ requires escrow $\geq f(\Delta,\theta)$).
3. **A mechanism with proofs**: an escrow/penalty schedule and bid rules such that (i) truthful bidding + completion is a Bayesian (or dominant-strategy) equilibrium, (ii) abandonment is exactly priced (the option has zero net subsidy), (iii) a quantified privacy guarantee (e.g., differential-privacy-style bound on what bidders learn about the order), with matching or near-matching bounds.
4. **A construction + simulation** demonstrating the tradeoff curve empirically on realistic finality parameters (Ethereum L1 vs. L2 rollups) — weaker than proofs but acceptable for the applied variant.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/8f659cf6-9566-4f49-b0ce-c483f7331541?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000303.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000303)
