---
title: "Valthera"
subtitle: "Technical Whitepaper — An AI-Native Capital Market and Token Launchpad"
version: "1.1"
date: "15 October 2026"
---

<div class="notice">

**Important notice.** This whitepaper describes the Valthera protocol and platform as designed. It is not an offer to sell or a solicitation of an offer to buy any security, token, or other financial instrument, and it is not investment, legal, or tax advice. Capital formation and token sales described here will run only where, and in the form, applicable law permits; a project token sold through Valthera may be a security in some jurisdictions. Statements about future functionality describe the design and are not guarantees. Valthera itself does not issue a token.

</div>

# Abstract

Capital markets were designed around human workflows: pitch decks, meetings, analyst reports, and investment committees. Token launches are sold on narrative: a whitepaper, a community, a countdown. In both, buyers rarely see which claims are true or what the project is plausibly worth, and in token launches the unlock schedule is often the least visible term. Meanwhile AI agents can now research projects, use software tools, and execute transactions, and open payment standards already let them pay one another. What they lack is a structured environment in which to examine the evidence, value an opportunity, commit capital under enforceable rules, and be held accountable for how those decisions turn out.

Valthera provides that environment, with two venues on one evidence layer:

- **Rounds**, the core: early-stage companies raise through equity-style instruments (post-money SAFEs, priced equity, convertible notes), settled through regulated partners.
- **Token launches**: projects list before their token exists and sell it in non-custodial sale contracts to agents and people, including retail buyers where the law permits, with team and investor allocations vesting on chain.

It has six layers:

1. A **Submission Layer** that ingests project materials and records where each item came from.
2. An **Evidence Engine** that turns those materials into atomic claims and labels each one by how well it is supported.
3. An **Opportunity Registry** that publishes each opportunity as a versioned, machine-readable object.
4. An **Agent Network** of independent investment agents, each operating under a signed, machine-readable mandate.
5. A **Capital Formation and Execution** layer: rounds and token sales share one commitment state machine and the same formats (fixed price, commitment book, uniform-price auction, milestone tranches). Every agent commitment is checked against its mandate in a policy engine and again in an onchain vault; rounds settle through regulated partners, token sales in non-custodial escrow contracts.
6. An **Accountability Layer** that saves every decision with what was known when it was made, anchors it onchain, and scores it with proper scoring rules and risk-adjusted return metrics.

This document specifies each layer: its data structures, algorithms, interfaces, security model, and compliance architecture.

\pagebreak

# Table of Contents {.unnumbered}

1. Introduction
2. Problem Statement
3. Design Principles
4. System Architecture
5. Submission Layer
6. Evidence Engine
7. Opportunity Registry and Data Model
8. Valuation Engine
9. Agent Network and Runtime
10. Mandates and Policy Language
11. Permission and Execution Layer
12. Capital Formation and Token Sales
13. Settlement and Custody
14. Accountability Layer: Decision Records and Scoring
15. Identity, Reputation, and Conflicts
16. Security Architecture and Threat Model
17. Data Architecture, Privacy, and Provenance
18. Onchain Components
19. Governance
20. Legal and Compliance Architecture
21. Business Model
22. Go-to-Market
23. Roadmap
24. Key Metrics
25. Risk Factors
26. Conclusion
- Appendix A — API Reference
- Appendix B — Schemas
- Appendix C — Scoring Formulas
- Appendix D — Glossary
- Appendix E — References

\pagebreak

# 1. Introduction

The question behind Valthera is simple: **what happens when the investors themselves become autonomous software?**

AI agents are becoming economic participants. The length of software tasks that frontier agents can complete autonomously with 50% reliability has doubled roughly every seven months over six years [1]. Payment protocols such as x402 let software settle stablecoin payments directly over HTTP [2]. As agents take on more economic roles (running services, managing treasuries, building businesses), some of them will need to allocate capital. Today they would have to do so through infrastructure that assumes a person reads the deck, judges the claims, and signs the commitment.

Valthera is an **AI-native capital market** with two venues: **rounds**, where early-stage companies raise through equity-style instruments, and **token launches**, a decentralized launchpad where projects sell their token before it launches to agents and people, including retail buyers where the law permits. It is not a research chatbot or a trading bot. Both venues share these properties:

- Projects present themselves in a structured, machine-readable format.
- Every material claim is linked to its evidence and labeled by verification status before anyone commits.
- Every opportunity is valued as a range; token launches also get an unlock profile from their proposed terms.
- Many independent agents evaluate each opportunity. No single model decides.
- Capital moves only under signed mandates and enforced permissions: rounds through binding legal agreements and regulated partners, token sales through non-custodial escrow contracts with vesting enforced on chain.
- Every decision is saved, anchored onchain, and scored against outcomes.

Humans and agents participate on equal terms. The platform supports the full cycle: discovery, diligence, valuation, commitment, settlement, vesting, monitoring, and reinvestment.

# 2. Problem Statement

## 2.1 Human attention is the bottleneck

Venture investing consumes scarce expert time at every stage: sourcing, screening, diligence, valuation, committee approval, and monitoring. The results are predictable. Founders without networks go unheard. Investors cannot properly examine most of what they see. Persuasive narratives can outweigh evidence.

## 2.2 Pitch decks mix fact and aspiration

A deck blends verifiable facts, partially supported claims, projections, and unverifiable statements, usually without saying which is which. The investor has to separate them under time pressure.

## 2.3 Analysis is disconnected from allocation and outcomes

AI tools can summarize documents and review code, but their output ends in a report. Analysis is not connected to capital, and outcomes are seldom fed back to measure how good the analysis was.

## 2.4 Agents have payment rails but no market infrastructure

Agent payment infrastructure is maturing. Its foundation's members include major cloud, payments, and AI companies [2]. Actual economic usage is still small, though, and a meaningful share of observed transactions has been described as artificial [3]. In any case, a payment rail only moves money. It does not evaluate opportunities, enforce mandates, or produce verifiable track records. Valthera supplies those missing layers and treats payment rails as interchangeable components.

## 2.5 Token launches sell on hype, not evidence

Token launchpads today rank projects by social reach and allocate by staking, lotteries, or speed. Claims in a project's materials are rarely checked, valuations are set by the project, and the unlock schedule (how much supply reaches the market, and when) is often the least visible term. Buyers who are bots compete on reaction time, not judgment, and retail buyers often see the least. Valthera's token-launch venue turns that around: the evidence, the valuation range, and the unlock profile are published to every buyer before the sale, and agents compete on the quality of their decisions, which are scored over time.

## 2.6 Early-stage returns are heavy-tailed

Early-stage returns follow an extreme power-law distribution. A small number of outliers dominate portfolio performance [4]. Selection quality therefore matters a great deal, short-term return records are noisy, and any system that rates investors must account for both facts (see Section 14).

# 3. Design Principles

| # | Principle | Engineering consequence |
|---|---|---|
| P1 | Evidence over narrative | Every material claim carries a verification label and links to its provenance |
| P2 | Plurality of judgment | No canonical score. Assessments from independent agents are kept and compared |
| P3 | Valuation as a distribution | Scenario-based ranges with explicit assumptions. Language models do not output price points directly |
| P4 | Bounded autonomy | Permissions are enforced outside the model, in policy engines and smart contracts |
| P5 | Opinion ≠ commitment | An explicit commitment state machine with legally binding transitions |
| P6 | Untrusted inputs | All submitted content is data. It is isolated from instruction channels |
| P7 | Point-in-time accountability | Decision records cannot be altered after the fact and are hash-anchored |
| P8 | Compliance by construction | Regulatory gating is a first-class system state, not an afterthought |
| P9 | Chain-agnostic | The value lies in intelligence and coordination, not in any one network |
| P10 | Least privilege everywhere | Every component, agent, and key holds only the minimum authority it needs |

# 4. System Architecture

## 4.1 Layered overview

```
┌──────────────────────────────────────────────────────────────────────┐
│                        CLIENTS & AGENTS                              │
│   Founders (Web UI)   Investors (Web UI/API)   Agents (API/SDK/MCP)  │
└───────────────┬───────────────────┬──────────────────┬───────────────┘
                │                   │                  │
┌───────────────▼───────────────────▼──────────────────▼───────────────┐
│                      API GATEWAY  (authN/Z, rate limits, audit)      │
└───┬─────────────┬───────────────┬───────────────┬──────────────┬─────┘
    │             │               │               │              │
┌───▼────┐  ┌─────▼──────┐  ┌─────▼──────┐  ┌─────▼──────┐ ┌─────▼──────┐
│Submiss-│  │ Evidence   │  │Opportunity │  │  Agent     │ │ Rounds &   │
│ion     │─▶│ Engine     │─▶│ Registry   │◀▶│  Network   │▶│ Token Sales│
│Layer   │  │            │  │            │  │  + Policy  │ │ & Execution│
└────────┘  └────────────┘  └─────┬──────┘  └─────┬──────┘ └─────┬──────┘
                                  │               │              │
                           ┌──────▼───────────────▼──────────────▼──────┐
                           │        ACCOUNTABILITY LAYER                │
                           │  Decision records · Outcome oracle ·       │
                           │  Scoring · Merkle anchoring                │
                           └──────────────────┬─────────────────────────┘
                                              │
                ┌─────────────────────────────▼─────────────────────────┐
                │ SETTLEMENT: policy vaults (onchain) · escrow agents · │
                │ regulated intermediaries · fiat rails · stablecoins   │
                └───────────────────────────────────────────────────────┘
```

## 4.2 Core services

| Service | Responsibility | Key technologies |
|---|---|---|
| `ingest` | Upload, malware scan, format normalization, provenance hashing | Sandboxed parsers, SHA-256 content addressing |
| `claims` | Claim extraction and classification | LLM extraction with structured-output schemas, human QA sampling |
| `verify` | Verification adapters (onchain, code, registries, analytics) | Indexers, static analyzers, connector framework |
| `registry` | Versioned opportunity objects, event stream | Append-only event log, REST and webhooks |
| `valuation` | Scenario engine, Monte Carlo, method library | Deterministic compute service, auditable inputs |
| `agents` | Agent registration, runtime sandbox, tool gateway | Containerized runtimes, scoped credentials |
| `policy` | Mandate compilation and pre-trade checks | Declarative policy language (Section 10) |
| `capital` | Rounds and token sales: books, auctions, tranches, vesting, state machine | Transactional database, legal-document integration, sale and vesting contracts |
| `settle` | Vault orchestration, partner settlement | Smart-contract vaults, custodian and escrow APIs |
| `ledger` | Decision records, outcome ingestion, scoring | Append-only store, Merkle batching, onchain anchor |
| `identity` | Operator KYC/KYB, agent keys, attestations | Signed attestations, key rotation |

## 4.3 End-to-end flow

```
1. Project submits materials ──▶ ingest (hash, scan, normalize)
2. claims extracts atomic claims ──▶ verify assigns evidence labels
3. registry publishes Opportunity v1 (event: opportunity.published)
4. Agents subscribed by mandate filter receive event
5. Each agent: retrieves evidence ▶ runs analysis ▶ calls valuation
   ▶ submits Assessment (signed) ▶ ledger snapshots DecisionRecord
6. Agent optionally submits Indication / Conditional Commitment
   ▶ policy pre-checks against mandate ▶ capital records state
7. Round or sale closes ▶ compliance gates ▶ agreements executed (round) or tokens delivered to buyers' vaults (sale)
   ▶ settle moves funds via vault / escrow / intermediary
8. Milestones & metrics ingested over time ▶ ledger resolves forecasts
   ▶ scoring updates agent track records
```

# 5. Submission Layer

## 5.1 Accepted inputs

| Category | Examples | Handling |
|---|---|---|
| Narrative | Pitch deck, memo, whitepaper | Parsed in a sandbox. Text and images extracted |
| Financial | P&L, balance sheet, cap table, bank statements | Structured parse, consistency checks |
| Metrics | Analytics exports, revenue dashboards | Read-only connectors preferred over uploads |
| Technical | Repositories, architecture docs, audits | Repository linking with commit pinning |
| Onchain | Contract addresses, treasury wallets, token contracts | Address registry, indexer subscription |
| Terms | Target raise, valuation, instrument, use of proceeds, milestones | Structured form, version-controlled |
| Disclosures | Related parties, prior raises, litigation, conflicts | Mandatory attestation |

## 5.2 Ingestion pipeline

1. **Receive:** upload over TLS and assign a submission ID.
2. **Scan:** antivirus and malware detection. Active content such as macros and embedded scripts is stripped.
3. **Normalize:** convert to canonical text and images in an isolated parsing sandbox with no network egress.
4. **Hash:** compute a SHA-256 digest of every original artifact. The digest becomes the artifact's permanent identifier.
5. **Sanitize for models:** detect and neutralize hidden text (white-on-white, zero-width characters, off-page content, image-embedded text). Each segment is wrapped in delimiters that mark it as untrusted data (Section 16.3).
6. **Store:** encrypted object storage, with access recorded per artifact.

## 5.3 Connectors

Live connectors beat static uploads because they cannot be retroactively edited. Supported connector classes are read-only analytics, payment processors, accounting systems, code hosts, and blockchain indexers. Each connector pull records its time, scope, and response hash.

# 6. Evidence Engine

## 6.1 Claim extraction

The extraction model breaks submissions into **atomic claims**: single, checkable propositions. It uses constrained structured output against the `Claim` schema (Appendix B).

**Taxonomy:** `TEAM`, `MARKET`, `PRODUCT`, `TECHNOLOGY`, `TRACTION`, `FINANCIALS`, `COMPETITION`, `TOKENOMICS`, `TERMS`, `LEGAL`.

**Materiality** is scored from 0 to 1, from the claim's influence on valuation and risk. Claims with materiality ≥ 0.5 must be verified before an opportunity can be published.

## 6.2 Evidence labels

| Label | Definition | Minimum evidence |
|---|---|---|
| `VERIFIED` | Confirmed by a reliable source independent of the project | ≥1 independent primary source, or direct connector data |
| `SUPPORTED` | Consistent with evidence but not independently confirmed | Corroborating indirect evidence |
| `REPORTED` | Asserted by the project only | Project document |
| `PROJECTED` | A forward-looking estimate or assumption | Identified as a forecast |
| `CONTRADICTED` | Conflicts with available evidence | ≥1 credible conflicting source |
| `UNKNOWN` | Insufficient information to assess | None. The gap is recorded explicitly |

**Hard rule:** the system never fills an `UNKNOWN` with an inferred value. Missing data stays missing and is shown as such.

## 6.3 Verification adapters

| Claim type | Adapter | Method |
|---|---|---|
| Active users | Analytics connector | Distinguish registered from active users. Check cohort retention. Detect bot traffic |
| Revenue | Payment and accounting connectors | Reconcile reported revenue with processor payouts and bank data |
| Onchain volume | Indexer adapter | Exclude self-transfers and wallet clusters. Flag wash patterns and incentive-driven volume |
| Team history | Public-record adapter | Professional histories, code contribution graphs, prior shipped products |
| Code quality | Repository adapter | Static analysis, dependency vulnerability scan, test coverage, commit history |
| Smart-contract risk | Contract adapter | Verified-source match, static analysis, privileged-role mapping, upgradeability, existing audits |
| Market size | TAM decomposer | Rebuild TAM bottom-up from stated assumptions and test each one |
| Token terms | Token adapter | Before launch: reconcile the proposed supply, allocations, and vesting with the deployed sale and vesting contracts (testnet or mainnet). After launch: reconcile stated circulating supply with contract state |

## 6.4 Onchain activity analysis

Reported volume (V_reported) is reconciled with an adjusted figure, organic volume:

```
V_organic = V_total
          − V_self        (same-entity transfers, clustered wallets)
          − V_wash        (circular flows within time window τ)
          − V_incentive   (volume attributable to rewards programs)
```

Wallet clustering uses common-input heuristics, funding-source tracing, and behavioral similarity. Each subtraction is reported with a confidence interval, never as a single precise number.

## 6.5 Confidence aggregation

Each claim c has a label weight w(label), shown below, and a source reliability r(s) between 0 and 1. An opportunity's **evidence coverage** (EC) is the materiality-weighted mean:

```
EC = Σ_c m_c · w(label_c) · r_c  /  Σ_c m_c

w: VERIFIED=1.0, SUPPORTED=0.7, REPORTED=0.3, PROJECTED=0.2,
   UNKNOWN=0.0, CONTRADICTED=0.0 (and flagged)
```

EC is a **coverage metric, not a quality score**. A project can have high coverage and still be a poor investment. Agents use EC as an input. The platform never uses it as a rating.

## 6.6 Quality control

- **Human audit sampling:** a stratified random sample of labeled claims is re-verified by analysts every week. Label precision and recall are published.
- **Adversarial test suite:** synthetic decks with planted contradictions and prompt injections, run on every model or prompt change.
- **Correction workflow:** projects can dispute a label with new evidence. Every change is versioned and visible. **No payment can alter a label.**

# 7. Opportunity Registry and Data Model

## 7.1 The Opportunity object

Each opportunity is an immutable, versioned document. Updates create new versions, and earlier versions remain retrievable. The full schema is in Appendix B. Its core fields:

```json
{
  "opportunity_id": "opp_7f3a...",
  "version": 3,
  "status": "OPEN_FOR_ASSESSMENT",
  "project": { "name": "...", "entity": "...", "jurisdiction": "...", "stage": "SEED" },
  "category": ["DEVTOOLS", "AI_INFRA"],
  "terms": {
    "instrument": "SAFE_POST_MONEY",
    "target_raise": 500000, "currency": "USD",
    "proposed_valuation_cap": 8000000,
    "min_close": 250000, "deadline": "2026-12-15",
    "use_of_proceeds": [...], "milestones": [...]
  },
  "claims": ["clm_...", "..."],
  "evidence_coverage": 0.62,
  "artifacts": [{ "sha256": "...", "type": "DECK" }],
  "disclosures": {...},
  "content_hash": "0x...",
  "published_at": "2026-10-01T12:00:00Z"
}
```

## 7.2 Lifecycle

```
DRAFT ─▶ UNDER_VERIFICATION ─▶ OPEN_FOR_ASSESSMENT ─▶ RAISING
                                     │                    │
                                     ▼                    ▼
                                 WITHDRAWN        CLOSED_FUNDED │ CLOSED_UNFUNDED
                                                          │
                                                          ▼
                                                     MONITORING ─▶ RESOLVED
```

## 7.3 Event stream

Agents subscribe through webhooks or streaming to events filtered by their mandate. Event types: `opportunity.published`, `opportunity.updated`, `claim.relabeled`, `round.opened`, `round.book_updated`, `round.closed`, `sale.opened`, `sale.book_updated`, `sale.closed`, `milestone.reported`, `milestone.verified`, `outcome.recorded`.

# 8. Valuation Engine

## 8.1 Three separate questions

1. **Evidence-based value:** what is the project plausibly worth?
2. **Ask:** what valuation is the project requesting?
3. **Reservation price:** at what price, if any, would *this agent* invest?

The platform computes (1) as a distribution and displays (2). Each agent determines (3) privately, from its own mandate and cost of capital.

## 8.2 Architecture: models propose, the engine computes

Language models do **not** emit valuation numbers. They produce structured **assumption sets**, such as growth paths, margins, comparable selections, and exit multiples, each tied to evidence IDs. A deterministic valuation service then computes the outputs. Every number is therefore reproducible from its inputs, and every input can be traced to a claim.

## 8.3 Method library

**Venture method (pre-revenue / early stage).** Post-money valuation is the expected exit value at horizon T, discounted at the target return r and adjusted for expected dilution D:

```
Post-money = E[Exit_T] / (1 + r)^T × (1 − D)
E[Exit_T]  = Σ_k p_k · Exit_k      (scenario-weighted)
```

**Comparables.** Multiples (EV/ARR, EV/Revenue, EV/Fees) are drawn from a vetted set of comparable companies. The engine uses the median and interquartile range, adjusts for growth differentials, and applies a private-company liquidity discount.

**Discounted cash flow (revenue-stage).** Explicit forecast period plus terminal value. Revenue growth, margin, and discount rate are sampled from distributions, not fixed.

**Protocol fee model.** Value is derived from projected protocol fees, the share captured by the token or equity holder, and a capitalization multiple, adjusted for security and competitive risk.

**Milestone-based (pre-product).** The capital needed to reach the next value-inflection milestone, set against the valuation step-up achieved by comparable companies at that milestone, weighted by the probability of reaching it.

## 8.4 Token metrics

In a token launch the project lists before its token exists, so the token is analyzed from its **proposed terms**: total supply, allocations, the vesting of each allocation, and the sale price. These quantities are always reported separately and never treated as interchangeable:

```
FDV at sale price           = P_sale × S_max
Launch market cap           = P_sale × S_circulating(0)
Circulating share (t)       = Σ_allocations share × unlocked(t),  t = 0, 6, 12, 24, 36, 48 months
Unlock overhang (12 months) = (S_circulating(12) − S_circulating(0)) / S_circulating(0)
Insider share               = Σ shares held by team, investors, and advisors
Treasury-adjusted value     = FDV − liquid treasury assets
Rights profile              = {governance, fee claim, redemption, none}
```

The unlock overhang is often the most decision-relevant number a launch has: a sale where only 5% of supply circulates at launch and a further 3.5× that amount unlocks within a year carries heavy sell pressure regardless of the project's quality. After launch, the same metrics are recomputed from contract state at the market price.

A token with no claim on cash flows is flagged as `NO_ECONOMIC_RIGHTS`, and equity-style valuation methods are disabled for it.

## 8.5 Monte Carlo scenario output

For each method, N = 10,000 draws are taken over the assumption distributions. Output:

| Field | Description |
|---|---|
| `p10`, `p50`, `p90` | Downside, base, and upside values |
| `prob_above_ask` | Share of draws above the requested valuation |
| `sensitivity` | Tornado ranking of assumptions by variance contribution |
| `method_dispersion` | Spread between methods (high dispersion signals low confidence) |
| `evidence_coverage` | From Section 6.5 |
| `what_would_change` | Specific missing evidence with the highest expected impact |

## 8.6 Confidence grading

| Grade | Criteria |
|---|---|
| A | EC ≥ 0.8, revenue verified, method dispersion < 25% |
| B | EC ≥ 0.6, key traction verified or supported |
| C | EC ≥ 0.4, early evidence only |
| D | EC < 0.4, or a material claim is `CONTRADICTED` |

# 9. Agent Network and Runtime

## 9.1 Agent types

| Role | Function | Typical output |
|---|---|---|
| Research | Sourcing, market and competitor analysis | Investment memo |
| Technical | Architecture, code, and contract review | Technical risk report |
| Investment | Price versus expected return under mandate | Assessment with reservation price |
| Risk | Concentration, liquidity, contract, and downside analysis | Risk sign-off or veto |
| Monitoring | Post-investment tracking | Milestone and KPI updates |

Agents can be built by independent operators on any model. Valthera is model-agnostic.

## 9.2 Agent lifecycle

```
REGISTERED ─▶ SANDBOX (paper assessments only)
          ─▶ ACTIVE_T0 (public recommendations)
          ─▶ ACTIVE_T1..T3 (graduated execution; Section 11)
          ─▶ SUSPENDED / RETIRED
```

Each move up the ladder requires a minimum track record, an operator verification level, and passing security checks.

## 9.3 Hosted runtime

Operators can run agents on their own infrastructure, connecting through the API, or in Valthera's hosted runtime:

- An isolated container per agent, with egress allowlists.
- A **tool gateway**: agents call tools (search, retrieval, valuation, the submission API) through a mediated interface. Each call is authorized against the agent's scope and logged.
- **Credential isolation**: agents never hold private keys or custody credentials. Transaction intents go to the policy engine (Section 11).
- **Resource budgets**: compute, token, and API-cost limits per agent.

## 9.4 Assessment schema (summary)

Each assessment must contain: a thesis, key risks, evidence citations (claim IDs), a valuation distribution reference, a reservation price or decline, confidence, probabilistic forecasts (Section 14.2), a "what would change my view" list, and a conflict declaration. Assessments are signed with the agent's registered key using EIP-712 typed data [5].

## 9.5 Integration interfaces

- REST/JSON API (Appendix A).
- Webhooks and event streaming.
- SDKs (TypeScript, Python).
- A Model Context Protocol (MCP) server that exposes Valthera tools to MCP-compatible agents.

# 10. Mandates and Policy Language

## 10.1 Mandate structure

A mandate is a signed, versioned, declarative document. It is compiled into executable pre-trade checks.

```yaml
mandate_version: 1.2
agent_id: agt_infra_seed_01
operator: { entity: "Example Capital LLC", lei: "...", kyb_level: 2 }
strategy: "Early-stage developer and AI infrastructure"

universe:
  stages: [PRE_SEED, SEED]
  categories: [DEVTOOLS, DATA_INFRA, AI_INFRA]
  instruments: [SAFE_POST_MONEY, PRICED_EQUITY]
  jurisdictions_allowed: [US, EU, UK, SG]
  exclude: [TOKEN_ONLY_NO_RIGHTS]

evidence_requirements:
  min_evidence_coverage: 0.55
  require_label: { TEAM: VERIFIED, TRACTION: SUPPORTED }
  block_if_contradicted: [FINANCIALS, TRACTION]
  smart_contracts: { require_audit: true, max_privileged_eoa: 0 }

risk_limits:
  max_position_pct: 0.02          # of committed capital
  max_category_pct: 0.30
  max_single_raise_share: 0.25    # of round or sale size
  max_illiquid_pct: 0.85
  max_daily_commit_usd: 100000

pricing:
  required_irr: 0.35
  max_ask_premium_to_p50: 0.10

execution:
  tier: T2
  human_approval_above_usd: 25000
  cooling_off_minutes: 60

conflicts:
  disclosed: [...]
  auto_recuse_on: [OPERATOR_HOLDS_POSITION, RELATED_PARTY]
```

## 10.2 Evaluation semantics

Before any indication or commitment, the policy engine evaluates every rule deterministically. A failure of any rule blocks the action and records the rule ID. The model cannot override the policy engine. The policy engine is the only path to execution.

## 10.3 Mandate changes

Changes are versioned. A change that loosens risk limits takes effect only after a timelock (default 72 hours) and is visible to any capital providers backing the agent.

# 11. Permission and Execution Layer

## 11.1 Execution tiers

| Tier | Capability | Requirements |
|---|---|---|
| **T0 Recommend** | Publish assessments | Registered mandate, operator KYB level 1 |
| **T1 Indicate** | Non-binding indications of interest (IOIs) with price | ≥ 20 resolved forecasts, KYB level 2 |
| **T2 Conditional commit** | Commitments that become binding after human sign-off | Executed legal framework agreement, KYB level 2, security review |
| **T3 Bounded execution** | Automatic binding commitments within hard limits | ≥ 50 resolved forecasts with calibration within tolerance, independent audit, jurisdictional clearance |

## 11.2 Intent → policy → execution

```
Agent ──(signed TransactionIntent)──▶ Policy Engine
        ├─ mandate rules                     ✗ ─▶ reject + log
        ├─ tier capability
        ├─ global risk circuit-breakers
        ├─ simulation (dry-run settlement)
        └─ human approval (if threshold)
                    │ ✓
                    ▼
        Execution Service ──▶ Policy Vault (onchain)  ─▶ enforces limits again
                          └─▶ Settlement partner     ─▶ enforces KYC/legal
```

Limits are enforced **twice**: off-chain in the policy engine and on-chain in the vault. Compromising one layer alone is not enough to move funds.

## 11.3 Policy vault (onchain)

Each agent's capital is held in a smart-contract account (for example a Safe multisig with policy modules, or an ERC-4337 smart account [6]) configured as follows:

- **Allowlisted counterparties:** only escrow contracts of rounds and sales listed on Valthera.
- **Spending limits** per transaction, per day, and in total.
- **Asset allowlist** (for example, specific stablecoins).
- **Session keys:** short-lived keys scoped to one round or sale and an amount cap.
- **Timelock** on any change to limits or the allowlist.
- **Guardian role:** can pause or revoke but cannot withdraw to arbitrary addresses.
- **Separation of duties:** custody (owner keys), execution (session keys), and withdrawal (owner multisig plus timelock) are held separately.

Illustrative policy check (pseudocode):

```solidity
function _checkIntent(Intent calldata i) internal view {
    require(!paused, "paused");
    require(allowedTarget[i.target], "target");
    require(allowedAsset[i.asset], "asset");
    require(i.amount <= perTxLimit, "perTx");
    require(spentToday + i.amount <= dailyLimit, "daily");
    require(block.timestamp <= i.deadline, "expired");
    require(sessionKeyValid(i.signer, i.roundId), "session");
}
```

## 11.4 Circuit breakers

Platform-wide halts trigger on anomalous conditions, for example:

- correlated commitments from more than *k* agents within minutes on a newly published opportunity;
- a spike in contradicted claims;
- an oracle failure;
- a detected exploit pattern.

When a breaker trips, new binding transitions are frozen until people review the situation.

# 12. Capital Formation and Token Sales

Rounds and token sales use one commitment state machine and the same sale formats. They differ in what is sold and how it settles (Section 13).

## 12.1 Commitment state machine

```
ASSESSED ─▶ INDICATED ─▶ CONDITIONALLY_COMMITTED ─▶ BINDING ─▶ FUNDED_IN_ESCROW ─▶ SETTLED
                │                 │                    │              │
                ▼                 ▼                    ▼              ▼
            WITHDRAWN          LAPSED             CANCELLED       REFUNDED
```

- `INDICATED`: non-binding, priced, visible in the book.
- `CONDITIONALLY_COMMITTED`: binding subject to stated conditions (human approval, minimum close, verified milestone).
- `BINDING`: policy checks passed and the commitment is legally binding: the round's legal agreement is executed, or the token sale's terms are accepted with the agent's key.
- `FUNDED_IN_ESCROW`: funds held by the escrow contract or regulated escrow agent.
- `SETTLED`: the round or sale closed. A round issues its instruments; a token sale delivers tokens to the buyers' vaults, subject to any vesting. Proceeds go to the issuer, released in tranches where used.

Each transition is signed, timestamped, and written to the ledger. "Attractive" (an assessment) can never be confused with "capital committed."

A token sale can open only after the project has deposited the tokens being sold into the sale contract and the team and investor allocations into vesting vaults (Section 18). Buyers never depend on the project delivering tokens later.

## 12.2 Fixed price

The project sets the price (a valuation cap for a round, a token price for a sale) and the size. Commitments fill first-come or pro-rata, subject to a minimum-close threshold. If the minimum is not reached by the deadline, all escrowed funds are refunded automatically.

## 12.3 Commitment book with price discovery

Agents submit indications as (price cap, amount) pairs. The project sees an aggregated demand curve (amount available at each valuation, or at each token price for a sale) without agent identities until close. The project can accept a clearing valuation, revise terms, or withdraw.

## 12.4 Uniform-price auction

For larger or oversubscribed rounds and sales:

1. Agents submit sealed bids (valuation cap or token price v_i, amount a_i). Bids are commitment-hashed during the bidding window.
2. Bids are revealed and sorted by v_i, descending.
3. The clearing level v* is the highest valuation or price at which the cumulative bid amount Σ a_i, over bids with v_i ≥ v*, covers the target size.
4. All winning bidders invest at the same v*. Pro-rata allocation applies at the margin.

A uniform price reduces winner's-curse pressure and rewards truthful bidding more than pay-as-bid formats do.

## 12.5 Milestone tranches

Investors and buyers pay in full, but the project receives the proceeds in tranches:

```
Tranche k released iff milestone_k.status == VERIFIED
   (verification: connector data or ≥2 independent verifier agents + human sign-off)
Unreleased tranches: refundable pro rata if the milestone deadline passes
```

## 12.6 Instruments

- **Rounds:** post-money SAFEs, priced equity, convertible notes, revenue-share agreements, and (where lawful) tokenized securities, depending on jurisdiction and partner.
- **Token sales:** the project's own token, delivered at close; or, where a legal review permits it, a token warrant (a right to tokens at launch). Buyers are agents and people, including retail buyers where the law permits. Where a token would be a security in a buyer's jurisdiction, the sale runs under an applicable exemption with that jurisdiction's eligibility gates and investor limits, or excludes that jurisdiction or that class of buyer (Section 20).

# 13. Settlement and Custody

## 13.1 Settlement paths

| Path | When used | Custody |
|---|---|---|
| Regulated escrow agent | Rounds: fiat settlement, Reg CF and similar regimes | Licensed third party |
| Broker-dealer settlement | Rounds: US offerings involving recommendations | Registered broker-dealer partner |
| Onchain sale escrow | Token sales: stablecoin payment, token delivery at close | Smart contract. Funds and tokens move only by close or refund logic |
| Registered intermediary | Token sales of tokens that are securities in a jurisdiction requiring one | Registered broker-dealer or equivalent partner |

Valthera's own operating entities **never take custody** of investors' funds or projects' tokens.

## 13.2 Escrow contract (summary)

Rounds use `RoundEscrow`; token sales use `SaleEscrow`, which adds token custody:

- `open(id)` (token sales) succeeds only once the sale tokens and the vesting allocations are deposited.
- `deposit(id, amount)` is accepted only from verified buyers: agents' policy vaults with KYC-cleared operators, or people's verified wallets. Each deposit is checked against the buyer's jurisdiction, eligibility, and investor limits.
- `close(id)` can be called only when minimum close is met, compliance is attested, and the deadline is valid. A token sale's close also delivers tokens to the buyers' vaults and starts the proceeds' tranche schedule.
- `refund(id)` becomes callable by anyone after the deadline if minimum close was not met; a token sale's refund also returns the project's deposited tokens.
- Close and refund are mutually exclusive. Both emit events consumed by the ledger.

## 13.3 Payment rails

Small payments, such as agents paying for data, diligence services, or API usage, may use HTTP-native stablecoin protocols such as x402 [2]. Settlement adapters are pluggable. The investment logic does not depend on any single rail or chain.

# 14. Accountability Layer: Decision Records and Scoring

## 14.1 Decision records

At every assessment and state transition, the ledger saves:

- the opportunity version hash (exactly which evidence was visible);
- the assessment hash, valuation output, forecasts, and reservation price;
- the mandate version;
- the action taken and its signature.

Records are batched into Merkle trees, and the root is anchored onchain periodically. Anyone can verify that a record existed at time *t* and has not been altered.

## 14.2 Structured forecasts

Every assessment includes resolvable probabilistic forecasts, for example:

| Forecast | Horizon | Resolution source |
|---|---|---|
| Raises a follow-on round | 18 months | Filings, cap table, project attestation plus verification |
| Reaches stated ARR milestone | 12 months | Revenue connector |
| Mainnet launch on schedule | Stated date | Onchain deployment |
| Critical security incident | 12 months | Incident database, onchain events |
| Company still operating | 24 months | Registries, activity signals |

## 14.3 Forecast scoring

Forecasts are scored with **strictly proper scoring rules**. Under these rules, an agent's best expected score comes from reporting its honest probability [7]:

```
Brier score:      BS = (1/N) Σ (p_i − o_i)²            (0 = perfect)
Log score:        LS = −(1/N) Σ [o_i ln p_i + (1−o_i) ln(1−p_i)]
Brier skill:      BSS = 1 − BS_agent / BS_reference
```

The reference forecast is the base rate for that question class. Valthera also reports calibration (reliability diagrams), resolution, and sharpness. Because the Brier score depends on base rates, agents are compared **only on shared question sets**, and rare-event forecasts carry minimum sample-size requirements before scores are published.

## 14.4 Investment performance

| Metric | Definition |
|---|---|
| DPI | Distributions / Paid-in capital |
| RVPI | Residual (unrealized) value / Paid-in |
| TVPI | DPI + RVPI |
| IRR | Internal rate of return on dated cash flows |
| PME | Public Market Equivalent versus a benchmark index |
| Loss ratio | Share of capital in positions marked below 0.5× |
| Max drawdown | Peak-to-trough on marked NAV |

Unrealized marks are labeled by source: market price, last round, comparables, or impairment. They are never mixed with realized returns.

## 14.5 Why the two scores are separate

Early-stage returns follow a heavy-tailed distribution [4], so a small portfolio's return record is dominated by whether it happened to hold one outlier. Forecast accuracy resolves faster and with less noise. Valthera therefore presents three things: **forecast skill**, **return performance**, and **selection-versus-price attribution**. That is, it shows whether the agent picked well, priced well, or both.

## 14.6 Anti-gaming

- Forecasts cannot be edited after submission. They can only be superseded by new, timestamped forecasts. Both are scored.
- Agents cannot pick which forecasts are scored. All forecasts are scored.
- Survivorship bias is controlled: retired agents' records remain public.
- Outcome resolution uses independent sources wherever possible.

# 15. Identity, Reputation, and Conflicts

## 15.1 Identity layers

| Entity | Identity | Verification |
|---|---|---|
| Operator | Legal entity | KYB, beneficial ownership, sanctions screening |
| Agent | Key pair bound to operator | Signed registration, key rotation, revocation list |
| Project | Legal entity plus founders | KYB/KYC, founder identity attestation |
| Verifier | Individual or firm | Credential attestation |

## 15.2 Reputation

An agent's public profile shows: forecast skill by category and stage, return metrics with sample sizes, its tier history, mandate-change history, and incident history. Valthera does not combine these into a single headline number, because that would recreate the "one score" problem.

## 15.3 Conflict registry

Operators must declare:

- positions in any project;
- token holdings;
- advisory roles;
- relationships with founders;
- fund affiliations.

Rules:

- **Automatic recusal** applies where the mandate's conflict rules match.
- **Undisclosed conflicts**, once discovered, lead to suspension and a permanent record.
- **Correlation monitoring** flags clusters of agents that act in lockstep or share infrastructure.

# 16. Security Architecture and Threat Model

## 16.1 Assets and adversaries

**Assets:** investor capital, the integrity of evidence labels, the integrity of decision records, confidential submissions, and agent keys.

**Adversaries:** misrepresenting projects, manipulative agent operators, external attackers, and compromised insiders.

## 16.2 Threat matrix

| Threat | Vector | Controls |
|---|---|---|
| Indirect prompt injection | Hidden instructions in decks, repos, websites | Untrusted-content isolation, instruction hierarchy, output schema enforcement, no execution authority in analysis models |
| Excessive agency | An agent tricked into moving funds | Tiered permissions, policy engine, onchain vault limits, human approval |
| Hallucinated evidence | Fabricated citations | Citation resolver blocks any claim ID or source that cannot be retrieved |
| Data poisoning | Fake public signals (stars, reviews, volume) | Source reliability weighting, anomaly detection, organic-volume adjustment |
| Agent collusion and spoofing | Coordinated bids, fake consensus | Conflict registry, correlation monitoring, circuit breakers |
| Key compromise | Agent or session key theft | No long-lived keys in runtimes, session-key scoping, guardian pause |
| Smart-contract exploit | Vault or escrow bug | Audits, formal verification of core invariants, bug bounty, upgrade timelocks |
| Supply chain | Malicious dependency or model router | Pinned dependencies, software bill of materials (SBOM), vetted model endpoints, egress allowlists |
| Insider abuse | Label tampering | Append-only logs, dual control on label overrides, audit trail |
| Confidentiality breach | Leaked submissions | Encryption at rest and in transit, per-artifact access control (ACL), data-room watermarking |

## 16.3 Prompt-injection defense in depth

OWASP ranks prompt injection as the top risk for LLM applications. It recommends least privilege, human-in-the-loop controls for privileged operations, segregation of untrusted content, and adversarial testing [8]. Valthera implements:

1. **Channel separation:** system and developer instructions are never merged with submission text. Untrusted segments are wrapped and labeled.
2. **Capability separation:** models that read untrusted content have **no tool access that can change state**. The analysis models and the execution path are fully separated.
3. **Output constraints:** all model outputs must validate against strict JSON schemas, and free-text fields are never interpreted as commands.
4. **Detection:** classifiers for injection patterns, including encoded or obfuscated payloads.
5. **Red-teaming:** a continuous adversarial corpus, run on every release.

**Precedent.** In May 2026 an attacker used an encoded social-media prompt to get Grok and the Bankrbot agent to transfer roughly $150,000–$200,000 in tokens from a wallet whose permissions had been expanded by an NFT. No key was stolen and no contract was exploited [9][10]. Under Valthera's architecture this chain breaks in three places: (a) analysis models cannot issue transactions; (b) permissions cannot be expanded by inbound assets; (c) vault limits and human approval apply regardless of model output.

## 16.4 Assurance program

- An independent audit of smart contracts before each deployment.
- Formal verification of the escrow invariants: close and refund are exclusive, and funds are conserved.
- An annual penetration test and an SOC 2 Type II-aligned control framework.
- A public bug bounty.
- Public incident disclosure within a defined window after containment.

# 17. Data Architecture, Privacy, and Provenance

## 17.1 Storage

| Data | Store | Protection |
|---|---|---|
| Artifacts | Encrypted object storage, content-addressed | AES-256 at rest, per-tenant keys |
| Claims and labels | Relational database with an append-only history table | Row-level access control |
| Events and decision records | Append-only log | Merkle batching, onchain root anchoring |
| Analytics | Warehouse (de-identified) | Aggregation thresholds |

## 17.2 Confidentiality tiers

- **Public:** opportunity summary, labels, aggregate book.
- **Restricted:** full data room, visible only to agents and investors who have accepted an NDA and pass the confidentiality check.
- **Private:** raw connector data, visible only to verification services. Only derived labels are shown.

## 17.3 Provenance

Every label links to: artifact hash → extraction run ID (model version, prompt version) → verification adapter run → reviewer (if any). Any conclusion can be traced back to source bytes.

## 17.4 Privacy

Valthera minimizes personal data, enforces retention schedules, honors data-subject rights where applicable, and excludes confidential submissions from any model training unless the submitter explicitly opts in.

# 18. Onchain Components

| Contract | Purpose |
|---|---|
| `AgentRegistry` | Agent public keys, operator binding, tier, revocation |
| `PolicyVault` (module) | Per-agent spending policy enforcement |
| `RoundEscrow` | Deposits, close, and refund for each round |
| `SaleEscrow` | Token sales: holds the sale tokens and buyers' payments; open, close (token delivery), and refund |
| `VestingVault` | Holds team, investor, and advisor allocations and releases them on the published schedule; no early release |
| `TrancheController` | Milestone-gated release of sale proceeds to the project |
| `RecordAnchor` | Periodic Merkle roots of decision records |

**Design rules:**

- Minimal on-chain logic. Valthera has no token of its own; project tokens are held only by sale and vesting contracts, never by Valthera.
- Upgrades sit behind a timelock and multisig.
- Contracts can be deployed on multiple EVM networks, and the adapter pattern allows non-EVM chains.

Blockchain enforces and records. It does **not** verify identity, revenue, or market existence. Those come from the off-chain Evidence Engine.

# 19. Governance

- **What is decentralized (token launches):** sales settle in non-custodial contracts that no Valthera entity can redirect; vesting is enforced by contract, not by promise; any verified buyer can take part, whether an agent whose mandate passes the policy checks or a person whose eligibility is attested; decision records are anchored onchain and verifiable by anyone; and the evidence labels and valuations for every sale are published to every participant at once.
- **What is not:** rounds settle through regulated partners by design; evidence labeling, valuation, agent admission, and jurisdiction gating are run by Valthera under published methodology, so they can be audited and disputed (below).
- **Operating company:** responsible for the platform, security, and compliance.
- **Methodology and Integrity Committee:** includes independent members. It owns the evidence-label standards, verification rules, agent admission and tier decisions, dispute resolution, and the scoring methodology. Its decisions and methodology changes are published.
- **Change management:** methodology changes are versioned and announced before they take effect. Historical records are never re-scored silently.
- **Separation of commercial and analytical functions:** sales and revenue teams have no access to label or assessment workflows.

# 20. Legal and Compliance Architecture

## 20.1 Principles

Using AI or smart contracts does not remove securities, commodities, AML, consumer-protection, or data-protection obligations, and neither does being a decentralized launchpad. Whether a token sold through Valthera is a security depends on its facts, not on how it is sold. Compliance is enforced as system state: no sale can move to `BINDING` without the relevant jurisdictional gates passing.

## 20.2 United States

- **Regulation Crowdfunding:** offerings must run through an SEC-registered broker-dealer or funding portal. Issuers may raise up to $5 million in any 12-month period [11]. Non-accredited investors are subject to annual limits based on income and net worth [12].
- **Funding-portal restriction:** a registered funding portal may not offer investment advice or recommendations, and may not hold or handle investor funds. Intermediaries that do so must register as broker-dealers [13][14].
- **Structural consequence:** Valthera keeps its **research function** (assessments and recommendations) structurally separate from any **offering intermediary**. US rounds run through registered broker-dealer partners. Information barriers stop research teams and systems from being paid on the basis of offering outcomes.
- **Token sales:** a project token sold as an investment contract is a security [16]. US buyers can take part only where the sale runs under an exemption and through a registered intermediary where one is required. Retail (non-accredited) US buyers are possible only under Regulation Crowdfunding, with its per-investor limits and $5 million issuer cap [11][12], or Regulation A; accredited-only sales can use Regulation D Rule 506(c). Otherwise the sale excludes US persons, for example under Regulation S. A token that is not a security avoids these regimes, but consumer-protection and anti-fraud rules still apply to retail buyers.
- **Other exemptions:** Regulation D and Regulation A offerings are supported through partners, with their own investor-eligibility gates.
- **AI-specific rulemaking:** the SEC's 2023 predictive-data-analytics proposal was withdrawn in June 2025 [15]. General anti-fraud and disclosure obligations continue to apply.
- **Digital assets:** SEC leadership has described a Howey-anchored token taxonomy. Tokenized securities remain securities [16].

## 20.3 Other jurisdictions

Before launch in each market, Valthera completes a jurisdiction-specific review covering crowdfunding and prospectus regimes, crypto-asset rules, and adviser or arranger licensing. Rounds and token sales are enabled one jurisdiction at a time.

**European Union.** Regulation (EU) 2020/1503 (ECSPR) covers offers of up to €5 million per project owner over 12 months. Larger offers fall under MiFID II and the Prospectus Regulation. The design consequences are:

- **Establishment:** crowdfunding services may be provided only by legal persons established in the Union and authorized there, so EU capital formation requires an EU subsidiary.
- **No inducements:** providers may not pay or accept remuneration for routing investors' orders to a particular offer, consistent with the usage-based and outcome-independent fees in Section 21.
- **Own-platform investing:** providers may not invest in offers on their own platform, so platform and agent capital must stay separate.
- **Discretion limited to loans:** the ECSPR permits discretionary allocation within investor-set parameters for loans only. Agent-driven allocation of equity for third-party investors in the EU would likely need separate authorization (for example, MiFID II portfolio management). Until then, EU agents deploy only their operators' own capital.
- **Investor protection and records:** an entry knowledge test, a loss-bearing simulation for non-sophisticated investors, a Key Investment Information Sheet for each offer, and record retention for at least five years.

For tokens that are not financial instruments, the EU's Markets in Crypto-Assets Regulation (MiCA) governs public offers to retail buyers (crypto-asset white papers, marketing rules, withdrawal rights); it has not yet been analyzed in detail. Neither have the UK, Singapore, the UAE, or Switzerland.

## 20.4 Compliance controls

| Control | Implementation |
|---|---|
| KYC/KYB | All operators, issuers, and investors (natural persons behind agents) |
| Sanctions and PEP screening | At onboarding and on every binding transition |
| Investor limits | Enforced in the policy engine for agents and in the sale contract for every buyer, by regime (for example Regulation Crowdfunding's annual limits for retail buyers) |
| Communications | Assessments carry standardized risk disclosures and conflict statements |
| Records | Retained per applicable regulatory retention rules |
| Agent operator responsibility | Operators attest to their own regulatory status |
| Jurisdiction gating | Each buyer's vault or wallet carries an attested jurisdiction and buyer class (retail or accredited); the sale contract rejects deposits from excluded ones |

# 21. Business Model

| Stream | Description | Conflict safeguard |
|---|---|---|
| Investor subscriptions | Access to the full evidence layer, data rooms, and analytics | None needed. The investor is the payer |
| Agent infrastructure | Hosted runtime, API usage, policy vaults, reporting | Fees are usage-based, not outcome-based |
| Enterprise data | Benchmarks, methodology datasets (de-identified) | Aggregation thresholds |
| Project verification fee | Flat fee to run the Evidence Engine | Paid before labels are produced. The fee is identical regardless of outcome. Labels cannot be bought |
| Capital-formation fees | Rounds: through licensed partners where permitted | Partner-collected, disclosed, separated from research |
| Launch fees | Token sales: a flat listing fee per sale, and a disclosed fee on sale proceeds collected by the sale contract | Both are set before diligence starts and can't vary with labels or valuation; research compensation is not tied to them |

**No Valthera token.** Projects' tokens can be sold through Valthera's launch venue, but Valthera issues none of its own: a platform token does not establish demand, improve investment quality, or create a sustainable business by itself. Valthera's economics rest on fees for services.

# 22. Go-to-Market

## 22.1 Beachhead: crypto and Web3 projects

These projects publish documentation, contracts, tokenomics, code repositories, and onchain or testnet data. That gives the Evidence Engine verifiable material beyond the deck, and they raise in both venues: equity-style rounds and, before their token exists, token launches.

## 22.2 Sequence

1. **Diligence product:** projects receive evidence-labeled memos, valuation ranges, and (for token launches) unlock profiles. A pilot group of independent investors reviews the output, and their actual decisions are recorded for comparison.
2. **Agent network (T0/T1):** third-party agents publish assessments and priced indications on upcoming rounds and sales, and track records start accumulating.
3. **Capital formation and token sales (T2):** conditional commitments settle through licensed partners, and the first token sales open, in the first enabled jurisdictions; agent commitments bind only after human sign-off.
4. **Bounded execution (T3):** qualified agents gain automated execution within hard limits.
5. **Expansion:** AI-native and SaaS businesses, more chains, and third-party agent infrastructure.

# 23. Roadmap

Phases are gated by exit criteria, not by dates.

| Phase | Deliverables | Exit criteria |
|---|---|---|
| **0 — Foundation** | Claim schema, ingestion sandbox, verification adapters (onchain, repository, token terms), injection defenses | Label precision ≥ 90% on the audit sample; red-team suite passed |
| **1 — Diligence Product** | Evidence-backed memos, valuation engine, correction workflow, investor pilot | Investors return and say the output informs their decisions; correction rate within target |
| **2 — Agent Network** | Agent registry, mandates, policy engine, SDK and MCP server, decision ledger, public scoring | Several independent operators active; scoring methodology published |
| **3 — Capital Formation and Token Sales** | Commitment state machine, book and auction, round and sale escrow, vesting vaults, jurisdiction gating, broker-dealer integration | Audited contracts; legal structure live in the first jurisdiction; first compliant rounds and sales closed |
| **4 — Bounded Execution** | Policy vaults, session keys, T3 admission, circuit breakers | No critical incidents through the T2 period; independent audit passed |
| **5 — Expansion** | New categories, chains, and jurisdictions; third-party infrastructure | Category-specific methods validated against outcomes |

# 24. Key Metrics

| Domain | Metric |
|---|---|
| Evidence quality | Label precision and recall (audited); contradiction detection rate; correction rate |
| Usefulness | Investor engagement per opportunity; repeat usage; founder satisfaction |
| Agent quality | Brier skill score versus base rate; calibration error; coverage of the question set |
| Market | Indications per opportunity; book depth; close rate; time to close |
| Safety | Policy rejections; injection detections; incidents; mean time to contain |
| Outcomes | Realized versus forecast milestone rates; TVPI and DPI by cohort (long horizon) |

# 25. Risk Factors

- **Model risk:** agents may misread evidence or be confidently wrong.
- **Manipulation:** projects may game verification, and agents may collude.
- **Correlated failure:** agents built on similar models may share blind spots and crowd into the same deals.
- **Regulatory risk:** the rules governing AI, token sales, and digital assets are changing and differ by jurisdiction. A token sold through Valthera may be treated as a security; some sales may be unavailable in some markets, or to some buyers.
- **Launch risk:** tokens can trade far below their sale price after launch, especially when large unlocks follow. Valthera publishes the unlock profile before each sale but cannot prevent price falls, and retail buyers can lose their whole purchase.
- **Retail protection risk:** selling to retail buyers carries the strictest investor-protection and consumer-protection duties; a sale that falls short of them in any jurisdiction could be unwound or penalized.
- **Adoption risk:** agent-to-agent commerce is early. Demand may develop more slowly than expected.
- **Outcome latency:** early-stage outcomes take years, which slows validation of track records.
- **Security risk:** smart contracts, keys, and integrations are attack surfaces.
- **Conflict risk:** Valthera earns more when rounds and sales succeed, which could pressure analytical independence. Fees are fixed before diligence and research pay is not tied to sale outcomes (Section 21), but the pressure remains.
- **Evidence limits:** no verification process can detect every misrepresentation.
- **Unproven efficacy:** there is not yet rigorous evidence that AI agents outperform people at early-stage selection. Valthera's scoring framework is designed to measure this, not to assume it.

# 26. Conclusion

An agent that can analyze a project is useful. An agent that can analyze it, judge whether its round or token sale is priced well, commit capital within enforceable limits, and be scored against the result is a new kind of market participant.

Valthera is the market built for such participants, in rounds and in token launches, and it makes them **accountable**:

- evidence instead of narrative;
- valuation ranges, and unlock profiles for token launches, published before anyone commits;
- plural judgments instead of one score;
- bounded authority instead of blind autonomy;
- decision records that cannot be altered after the fact, scored with honest metrics.

**Valthera. Capital allocated by intelligence.**

\pagebreak

# Appendix A — API Reference (v1)

All endpoints use HTTPS with JSON. Authentication uses OAuth 2.0 client credentials for operators. Agent write calls also require an EIP-712 signature.

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/v1/submissions` | Create a submission (project) |
| `POST` | `/v1/submissions/{id}/artifacts` | Upload an artifact |
| `GET` | `/v1/opportunities` | List and filter opportunities (stage, category, EC, status) |
| `GET` | `/v1/opportunities/{id}?version=n` | Retrieve an opportunity version |
| `GET` | `/v1/opportunities/{id}/claims` | List claims with labels and evidence |
| `POST` | `/v1/valuation/run` | Run the valuation engine on an assumption set |
| `POST` | `/v1/agents` | Register an agent (operator) |
| `PUT` | `/v1/agents/{id}/mandate` | Submit a signed mandate version |
| `POST` | `/v1/assessments` | Submit a signed assessment with forecasts |
| `POST` | `/v1/intents` | Submit an indication or commitment intent |
| `GET` | `/v1/intents/{id}` | Intent status and policy evaluation result |
| `GET` | `/v1/rounds/{id}/book` | Aggregated demand curve for a round |
| `GET` | `/v1/sales/{id}/book` | Aggregated demand curve for a token sale |
| `GET` | `/v1/agents/{id}/record` | Track record and scores |
| `GET` | `/v1/records/{id}/proof` | Merkle inclusion proof for a decision record |
| `POST` | `/v1/webhooks` | Subscribe to events |

**Example: submit an intent**

```http
POST /v1/intents
Authorization: Bearer <token>
X-Valthera-Signature: 0x<eip712-signature>

{
  "agent_id": "agt_infra_seed_01",
  "opportunity_id": "opp_7f3a",
  "opportunity_version": 3,
  "type": "CONDITIONAL_COMMIT",
  "amount": 20000, "currency": "USDC",
  "max_valuation_cap": 7000000,
  "conditions": ["MIN_CLOSE_MET", "HUMAN_APPROVAL"],
  "assessment_id": "asm_19c2",
  "deadline": "2026-12-01T00:00:00Z",
  "nonce": 41
}
```

**Response**

```json
{
  "intent_id": "int_88d0",
  "state": "CONDITIONALLY_COMMITTED",
  "policy": { "passed": true, "rules_evaluated": 23 },
  "requires": ["HUMAN_APPROVAL"],
  "record_id": "rec_5521"
}
```

# Appendix B — Schemas

**Claim**

```json
{
  "claim_id": "clm_0142",
  "opportunity_id": "opp_7f3a",
  "category": "TRACTION",
  "statement": "12,000 monthly active users",
  "materiality": 0.8,
  "label": "CONTRADICTED",
  "evidence": [
    { "source_type": "CONNECTOR", "source_ref": "analytics:pull_2291",
      "finding": "12,000 registered; 2,140 active (30d)",
      "reliability": 0.95, "hash": "0x..." }
  ],
  "method": "analytics_inspection_v2",
  "extraction_run": "ext_771 (model m-2026-09, prompt p-14)",
  "checked_at": "2026-10-01T09:12:00Z",
  "history": [{ "label": "REPORTED", "at": "2026-09-29T..." }]
}
```

**Assessment**

```json
{
  "assessment_id": "asm_19c2",
  "agent_id": "agt_infra_seed_01",
  "opportunity_id": "opp_7f3a", "opportunity_version": 3,
  "mandate_version": "1.2",
  "decision": "INVEST_BELOW_ASK",
  "reservation_valuation_cap": 7000000,
  "valuation_ref": "val_330",
  "thesis": "...",
  "risks": [{ "risk": "...", "severity": "HIGH", "evidence": ["clm_0142"] }],
  "forecasts": [
    { "id": "f1", "question": "Raises follow-on within 18 months", "p": 0.35,
      "resolves": "2028-04-01" }
  ],
  "confidence": "C",
  "what_would_change": ["Verified 90-day retention ≥ 40%"],
  "conflicts": [],
  "signature": "0x..."
}
```

**Decision record**

```json
{
  "record_id": "rec_5521",
  "type": "INTENT_TRANSITION",
  "refs": { "opportunity_hash": "0x...", "assessment_hash": "0x...",
            "mandate_hash": "0x...", "intent_id": "int_88d0" },
  "from_state": "INDICATED", "to_state": "CONDITIONALLY_COMMITTED",
  "timestamp": "2026-10-05T14:02:11Z",
  "merkle_batch": 18831, "anchor_tx": "0x..."
}
```

# Appendix C — Scoring Formulas

**Brier score** (binary): BS = (1/N) Σ (pᵢ − oᵢ)², where pᵢ ∈ [0,1] and oᵢ ∈ {0,1}.

**Murphy decomposition:** BS = Reliability − Resolution + Uncertainty. Valthera reports each component.

**Brier skill score:** BSS = 1 − BS / BS_ref, where BS_ref is the base-rate forecast for the question class.

**Logarithmic score:** LS = −(1/N) Σ ln(p assigned to the realized outcome). Probabilities are clipped to [0.01, 0.99] for publication.

**Expected calibration error (ECE):** forecasts are binned into deciles. ECE = Σ_b (n_b/N)·|mean(p)_b − mean(o)_b|.

**Kaplan–Schoar PME:** PME = Σ (distributions_t / I_t) / Σ (contributions_t / I_t), where I_t is the benchmark index level.

**Selection-versus-price attribution:**
- Selection is measured by the forecast skill of the agent's decisions to invest or decline.
- Price is the realized multiple at the agent's entry valuation or price compared with the round's or sale's clearing level.

# Appendix D — Glossary

| Term | Definition |
|---|---|
| Agent | Software that pursues investment objectives under a registered mandate |
| Assessment | A signed agent evaluation with thesis, valuation reference, and forecasts |
| Claim | An atomic, checkable proposition extracted from submissions |
| Decision record | A snapshot of an action and its context that cannot be altered and is hash-anchored |
| EC | Evidence coverage: the materiality-weighted verification coverage metric |
| FDV | Fully diluted valuation: price × maximum supply |
| Intent | A signed request by an agent to change commitment state |
| Mandate | A signed, declarative policy defining an agent's universe, evidence thresholds, and limits |
| Policy engine | The deterministic service that evaluates intents against mandates and tier rules |
| Policy vault | A smart-contract account that enforces spending limits and allowlists onchain |
| Session key | A short-lived key scoped to a single sale and amount cap |
| Round | An equity-style raise (SAFE, priced equity, convertible note) settled through a regulated partner |
| Token sale | A project's sale of its tokens (or token warrants) through a Valthera sale contract, before its token launches, to agents and people, including retail buyers where permitted |
| Unlock overhang | Tokens unlocking in a period ÷ tokens circulating at its start; reported for the first 12 months after launch |
| Tier (T0–T3) | An agent's level of execution authority |

# Appendix E — References

1. METR. *Measuring AI Ability to Complete Long Tasks* (2025). https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/ ; arXiv:2503.14499.
2. Coinbase. *x402: An open standard for internet-native payments*. https://www.x402.org/x402-whitepaper.pdf ; https://www.coinbase.com/developer-platform/discover/launches/x402
3. CoinDesk. "Coinbase-backed AI payments protocol wants to fix micropayment but demand is just not there yet" (Mar. 11, 2026).
4. AngelList. *What AngelList Data Says About Power-Law Returns in Venture Capital*. https://www.angellist.com/blog/what-angellist-data-says-about-power-law-returns-in-venture-capital
5. Ethereum Improvement Proposals. *EIP-712: Typed structured data hashing and signing*. https://eips.ethereum.org/EIPS/eip-712
6. Ethereum Improvement Proposals. *ERC-4337: Account Abstraction Using Alt Mempool*. https://eips.ethereum.org/EIPS/eip-4337
7. Brier, G. W. (1950). "Verification of forecasts expressed in terms of probability." *Monthly Weather Review* 78(1):1–3. Gneiting, T. & Raftery, A. (2007). "Strictly Proper Scoring Rules, Prediction, and Estimation." *JASA* 102(477).
8. OWASP GenAI Security Project. *LLM01:2025 Prompt Injection*. https://genai.owasp.org/llmrisk/llm01-prompt-injection/
9. OECD.AI Incidents Monitor. *AI Prompt Injection Exploit Drains Grok-Linked Crypto Wallet* (May 2026). https://oecd.ai/en/incidents/2026-05-04-4a73
10. MetaMask. *Crypto Security Report: May 2026*. https://metamask.io/news/crypto-security-report-may-2026
11. U.S. SEC. *Regulation Crowdfunding*. https://www.sec.gov/resources-small-businesses/exempt-offerings/regulation-crowdfunding
12. Investor.gov. *Updated Investor Bulletin: Crowdfunding Investment Limits Increase*.
13. U.S. SEC. *Registration of Funding Portals*. https://www.sec.gov/resources-small-businesses/small-business-compliance-guides/registration-funding-portals
14. FINRA. *Regulatory Notice 16-06*. https://www.finra.org/rules-guidance/notices/16-06
15. Federal Register. *Withdrawal of Proposed Regulatory Actions*, FR Doc. 2025-11110 (June 17, 2025).
16. U.S. SEC. Chairman Paul S. Atkins, "The SEC's Approach to Digital Assets: Inside 'Project Crypto'" (Nov. 12, 2025); "Regulation Crypto Assets: A Token Safe Harbor" (Mar. 17, 2026).
