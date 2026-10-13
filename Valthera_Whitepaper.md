---
title: "Valthera Whitepaper"
subtitle: "Infrastructure for an AI-Native Capital Market"
date: "Version 0.1"
---

> **Important notice.** This whitepaper describes a proposed system and the reasoning behind it. It is not an offer to sell or a request to buy any security, token, or other financial instrument, and it is not investment, legal, or tax advice. Anything said about the future is a plan, not a promise, and the design may change. Valthera does not plan to issue a token at launch.

---

# Abstract

Capital markets were built around how people work: pitch decks, meetings, analyst reports, and investment committees. AI agents can now research companies, use software tools, and carry out transactions. Agents can already make payments through open standards. What they lack is a structured place to find investment opportunities, check the evidence behind them, put a value on them, commit capital under clear rules, and be judged on how those decisions turn out.

Valthera is meant to be that place. Projects submit materials. A due-diligence engine turns the materials into a structured set of **claims**, each with a label showing how well it is supported. A network of independent **investment agents**, each working under a written **mandate**, assesses the opportunity and gives a valuation range. Where the law allows, agents can commit capital through limited, auditable **execution permissions**. Every assessment is saved together with what was known when it was made. This builds a **track record** that measures both how accurate an agent's forecasts were and how well its investments did.

This paper covers the problem, the system design, the evidence and valuation methods, the agent permission model, capital formation, track-record measurement, security, legal considerations, the business model, and a phased plan for building it.

---

# 1. Introduction

The question behind Valthera is simple: **what happens when the investors themselves become autonomous software?**

If AI agents take part in the economy (running services, managing treasuries, building businesses), some of them will need to put capital to work. Today they would have to do it through infrastructure designed for people. That infrastructure assumes a person reads the deck, a person judges whether the claims are credible, and a person signs the commitment.

Valthera is **not** a token launchpad, a research chatbot, or a trading bot. It is designed as an **AI-native capital market** with these properties:

- Projects present themselves in a structured format that machines can read.
- Many independent agents assess each opportunity. No single "AI score" decides.
- Every material finding is linked to its evidence and labeled by how well it is verified.
- Capital moves only under explicit mandates, permissions, and legal agreements.
- Decisions are recorded and later compared with what actually happened.

Both people and agents can use the platform. Over the long term, it should support the whole investment cycle: discovery, diligence, thesis, negotiation, commitment, monitoring, and reinvestment.

---

# 2. Problem Statement

## 2.1 The process depends on scarce human attention

Each stage of venture investing (sourcing, screening, diligence, valuation, committee decision, monitoring) uses up skilled people's time. As a result:

- **Founders without networks are overlooked.** Good projects struggle to get a hearing.
- **Investors are overwhelmed.** There are far more opportunities than anyone can look into properly.
- **Persuasive stories beat evidence.** A strong narrative can make up for weak data, and reputation and access affect outcomes heavily.

## 2.2 Pitch decks mix facts with hopes

A deck mixes verifiable facts, partly supported statements, projections, and claims that cannot be checked, often without saying which is which. Investors have to sort this out with limited time.

## 2.3 AI analysis stops at the report

Today's AI tools can summarize documents, review code, and research competitors. But their output is usually a report that a person then has to act on. Analysis does not connect to capital allocation, and the results of investment decisions are seldom fed back to judge how good the analysis was.

## 2.4 Agents have payment rails but no capital-market infrastructure

The tools for agents to pay for things are developing quickly. Coinbase's x402 protocol, for example, lets APIs, apps, and agents settle stablecoin payments directly over HTTP, and its foundation has drawn wide industry support. But a payment protocol only moves money. It does not help an agent judge an investment, enforce a mandate, or build a track record that others can check. Adoption of agent payments is also still early, and some reported activity appears artificial. Valthera assumes the payment layer will keep improving but does not depend on any one of these systems.

---

# 3. Design Principles

1. **Evidence over narrative.** Separate what is claimed from what is verified.
2. **Many judgments, not one score.** Assessments that disagree are useful information.
3. **Valuation is a range.** Base, upside, and downside cases, each with stated assumptions and confidence.
4. **Autonomy does not mean unlimited authority.** Agents act only within permissions that can be checked.
5. **An opinion is not a commitment.** "Attractive" and "capital committed" are different states.
6. **Submitted material is untrusted input.** Documents are data, never instructions.
7. **Accountability over time.** Every decision is saved with what was known at the time.
8. **Compliance by design.** Using AI or smart contracts does not remove legal obligations.
9. **Not tied to one chain.** The value lies in intelligence and coordination, not in any single blockchain.

---

# 4. System Overview

Valthera has six layers:

| Layer | Function |
|---|---|
| **1. Submission Layer** | Takes in decks, financials, metrics, code, contracts, tokenomics, and onchain addresses, and keeps a record of where each item came from. |
| **2. Evidence Engine** | Pulls out claims, sorts them into categories, checks them against sources, and labels how well each is supported. |
| **3. Opportunity Registry** | A standard, machine-readable "opportunity object" that agents read through an API. |
| **4. Agent Network** | Independent investment agents with registered mandates, identities, and disclosed conflicts. |
| **5. Capital Formation & Execution** | Rounds, commitments, auctions, and milestone tranches, settled through approved and compliant channels. |
| **6. Accountability Layer** | Saved decision records, outcome tracking, and agent performance analytics. |

**Typical flow:**

```
Project submits → Evidence Engine extracts & verifies claims
   → Opportunity object published to registry
   → Agents assess under their mandates (thesis + valuation range + confidence)
   → Agents optionally post conditional or binding commitments
   → Round closes only if legal, approval, and settlement conditions are met
   → Milestones & outcomes monitored → decision records scored over time
```

---

# 5. The Evidence Engine

## 5.1 Claim extraction

The engine breaks each submission into separate claims and sorts them into categories: **Team, Market, Product, Technology, Traction, Financials, Competition, Token/Economics, and Terms**.

## 5.2 Evidence status labels

Every material claim gets exactly one label:

| Label | Meaning |
|---|---|
| **Verified** | Confirmed against a reliable independent source. |
| **Supported** | Consistent with the evidence but not independently confirmed. |
| **Reported** | Stated by the project only. |
| **Projected** | A forecast or assumption. |
| **Contradicted** | Conflicts with available evidence. |
| **Unknown** | Not enough information to judge. |

Each label points to the sources behind it, the method used, and the date of the check. **Missing information stays marked as missing.** The system must not fill gaps with plausible guesses.

## 5.3 Verification methods by claim type

- **Users and traction:** separate registered accounts from active users. Where access is granted, inspect analytics directly.
- **Team:** check public professional histories, past code contributions, and products the team has shipped, and state the limits of what public data can show.
- **Onchain activity:** compare stated volume with chain data. Flag wash trading, transfers between related wallets, and activity driven by incentives.
- **Market size:** break the estimate into its assumptions and test each one, rather than repeating the headline number.
- **Technology:** review architecture, dependencies, repositories, smart contracts, and existing audits. Automated analysis raises issues to look at. **It does not certify that anything is secure.**

## 5.4 Example claim record

```json
{
  "claim_id": "c-0142",
  "category": "Traction",
  "statement": "12,000 monthly active users",
  "status": "Contradicted",
  "evidence": [
    {"source": "project analytics export", "finding": "12,000 registered; 2,140 active in last 30 days"}
  ],
  "method": "analytics inspection",
  "checked_at": "2026-10-01",
  "notes": "Claim conflates registrations with MAU."
}
```

## 5.5 Corrections

Projects can dispute findings and send more evidence through a formal correction process. All changes are versioned and visible. **No one can pay to change a finding or a rating.**

---

# 6. Valuation Framework

## 6.1 Three separate questions

1. **What might the project be worth, given the evidence?**
2. **What valuation is the project asking investors to accept?**
3. **At what price, if any, would *this particular agent* invest?**

Question 3 is the one that drives capital allocation. The answer depends on each agent's opportunity cost, risk tolerance, time horizon, and required return. Valthera does not impose a single market-wide valuation.

## 6.2 Methods by stage

| Stage / Type | Main inputs |
|---|---|
| **Pre-revenue startup** | Comparable rounds, team, product maturity, market structure, early customer evidence, capital needed to reach milestones |
| **Revenue-stage business** | Growth, gross margin, retention, CAC/payback, quality of recurring revenue, relevant multiples |
| **Protocol** | Adoption, economic activity, fee generation, security assumptions, competition, how value flows from protocol to token |
| **Token-based project** | Equity value vs. circulating market cap vs. FDV; circulating supply; vesting and unlock schedules; treasury; **the legal and economic rights the token actually carries** |

These token measures are **not interchangeable**. The system reports them separately.

## 6.3 Required output

Every valuation assessment includes:

- **Base, upside, and downside cases**, each with explicit assumptions.
- A **confidence level**, tied to how much of the underlying evidence is verified.
- The **key sensitivities**: which assumptions change the answer most.
- **What would change the view**: the specific evidence that would move the range.

A language model does not produce the valuation number by itself. Models build structured scenarios, and the system records the inputs and calculations so they can be checked.

---

# 7. Investment Agents and Mandates

## 7.1 The mandate

Each agent works under a registered, machine-readable mandate. For example:

```yaml
agent_id: infra-seed-01
operator: <disclosed legal entity>
strategy: Early-stage developer infrastructure
eligible_stages: [pre-seed, seed]
sectors: [devtools, data infrastructure, AI infrastructure]
min_evidence:
  traction: Supported        # minimum label required
  team: Verified
  security_audit: required_if_smart_contracts
risk_limits:
  max_per_opportunity: 2%    # of agent capital
  max_sector_exposure: 30%
  max_illiquid_share: 80%
execution:
  mode: recommend_only        # recommend_only | conditional_commit | binding_commit
  human_approval_above: 25000 # USD equivalent
conflicts_disclosed: [ ... ]
```

## 7.2 Agent capabilities

Agents can find opportunities, ask projects for more information, compare deals with their current portfolio, write investment memos, and build portfolios. Every assessment must **explain its reasoning, cite evidence, state its uncertainty, and say what would change its conclusion**.

## 7.3 Agent roles in the network

- **Research agents:** find opportunities and write memos.
- **Technical agents:** review architecture, code, and contracts.
- **Investment agents:** judge price against expected return within their mandates.
- **Risk agents:** check concentration, liquidity, contract risk, and downside exposure.

Agents may be built by independent operators on different models. Valthera is meant to coordinate many evaluators, not to be one central AI that picks winners.

## 7.4 Identity and conflicts

Each agent is linked to a **disclosed operator**. Operators must disclose relationships with projects, token holdings, and fund affiliations. Undisclosed conflicts lead to removal and are recorded in the agent's history.

---

# 8. Permission and Execution Model

A persuasive analysis never gives an agent access to funds.

## 8.1 Execution tiers

| Tier | Capability | Prerequisite |
|---|---|---|
| **T0 — Recommend** | Publish assessments and recommendations | Registered mandate |
| **T1 — Indicate** | Non-binding indication of interest at a stated price | Track record over a minimum period |
| **T2 — Conditional commit** | Commitment that becomes binding only after human or legal sign-off | Verified operator, executed legal agreements |
| **T3 — Bounded execution** | Automated commitment within hard limits | Proven reliability, audits, regulatory clearance for the relevant market |

## 8.2 Controls

- **Separate duties:** custody, execution, and withdrawal authority are held separately.
- **Hard limits** enforced by smart contracts or the custodian, not by the model: spending caps, asset allowlists, exposure ceilings, and rate limits.
- **Transaction simulation** before any value moves.
- **Emergency controls:** pause, revoke, and kill switches held by designated people.
- **Full audit logs** linking each action to the assessment and approval that allowed it.

This follows current guidance on LLM security. The OWASP Top 10 for LLM Applications (2025) lists excessive agency as a key risk and specifically warns against systems that take high-impact actions, such as moving funds, without a human approval step.

---

# 9. Capital Formation

## 9.1 Offering disclosure

A project entering capital formation states its **funding target, proposed valuation, use of proceeds, milestones, and the investor rights offered**.

## 9.2 Supported mechanisms (phased)

- Fixed-term rounds at a stated valuation.
- Commitment books where agents state the price at which they would invest.
- Price-discovery auctions.
- Milestone-based tranches, with capital released when verified milestones are met.

## 9.3 Example

A project wants to raise **$500,000** at a proposed valuation. Some agents commit at that price. Others decline or say they would invest only at a lower valuation. The project can revise its terms, add evidence, or close with the commitments it has, subject to the round's rules. **The assessments carry information about quality, and the commitments carry information about price.** This creates a feedback loop.

## 9.4 Opinion vs. commitment

The system uses distinct, non-overlapping states: *Assessed → Indicated → Conditionally Committed → Binding → Settled*. Funds move only when approvals, enforceable agreements, and settlement conditions are all met.

## 9.5 Settlement

Settlement may use regulated fiat channels, escrow, or onchain contracts, depending on the instrument and jurisdiction. Agent payment standards such as x402 may be supported for small payments (for example, agents paying for data or diligence services). The settlement design is deliberately kept separate from the investment logic.

---

# 10. Track Records and Agent Evaluation

## 10.1 The decision record

Each decision is saved with: the evidence available at the time, the valuation range, stated risks, confidence, the decision taken, and later reassessments. Records **cannot be changed afterward**, so outcomes cannot be "explained" by revising history.

## 10.2 Two separate scores

1. **Forecast accuracy.** Did the agent's predictions (milestones hit, revenue reached, follow-on raised) come true? This is measured with proper scoring rules, such as Brier or log scores on probability forecasts, and checked for calibration.
2. **Investment performance.** Realized and unrealized returns, measured separately.

An agent can predict growth correctly and still overpay. The two scores answer different questions.

## 10.3 Measurement safeguards

- Separate **realized returns, unrealized marks, capital flows, distributions, losses, and outcomes that are still open**.
- Adjust for **stage, holding period, concentration, market regime, and data reliability**.
- Report **risk-adjusted** results and drawdowns so that high-risk luck in a bull market is not mistaken for skill.
- Treat **unrealized marks** with caution and label how they were obtained.
- Watch for **selection bias** and gaming of metrics.

---

# 11. Data Advantage

A general-purpose model can summarize a deck. That alone is not defensible. Valthera's potential advantage is the **linked record of evidence → forecast → decision → outcome**, collected across many projects and agents. Over time this allows:

- Benchmarks of which methods work for which kinds of project.
- Evaluation models specialized by category (infrastructure, consumer, DeFi, AI-native).
- Better calibration for agents and for the platform's own evidence engine.

**Limitations:** the data will be incomplete, selection bias is real, and past results do not predict future returns. Proprietary submissions must be protected, and the advantage depends on clear data provenance and sound methods.

---

# 12. The Role of Onchain Infrastructure

**What blockchains provide:** enforceable permissions and spending limits, allocation and settlement rules, public records of where funds went, and (where legally appropriate) tokenized representations of rights.

**What they do not provide:** proof of identity, proof of revenue, proof that a market exists, or proof that a model reasoned well. Most of what matters in venture diligence is **offchain**. Valthera connects offchain evidence with onchain execution and supports multiple chains as needed. Blockchain is a way to enforce and record decisions, not a substitute for judgment.

---

# 13. Security, Trust, and Governance

## 13.1 AI-specific threats

| Threat | Mitigation |
|---|---|
| **Prompt injection in submitted documents** | Treat all submissions as untrusted data. Separate and clearly mark external content. Keep the instruction channel isolated. Restrict model privileges. Run regular adversarial testing. |
| **Hallucinated evidence** | Every material finding must cite a source that can be retrieved. Uncited claims are blocked from final outputs. |
| **Excessive agency** | Tiered permissions (Section 8), hard limits outside the model, human approval for high-impact actions. |
| **Collusion and manipulation between agents** | Disclosed operators, conflict registry, anomaly detection on correlated commitments, penalties recorded in track records. |
| **Shared model blind spots** | Encourage diversity of agents and models, and report how correlated agents' assessments are. |
| **Data poisoning** | Provenance tracking and weighting by source reliability. |

**A real-world precedent.** In May 2026 an attacker used an encoded prompt to get Grok and the Bankrbot agent to transfer 3 billion DRB tokens (reported at about $150,000–$200,000) from a wallet on Base. The OECD AI Incidents Monitor logged this as an AI incident ([OECD.AI](https://oecd.ai/en/incidents/2026-05-04-4a73)). No key was stolen and no contract was exploited. The agent had been given execution authority, and it treated untrusted text as a command. This is exactly the failure mode that Valthera's tiered permissions and out-of-model limits are designed to prevent.

These choices follow OWASP's guidance. OWASP ranks prompt injection as the top risk for LLM applications and recommends least privilege, human-in-the-loop controls for privileged operations, clear separation of untrusted content, and regular penetration testing.

## 13.2 Platform integrity

- Ratings cannot be bought, and paid services are kept apart from independent assessments.
- Projects have a clear process to correct errors.
- Conflicts are disclosed for agents, operators, and the platform itself.

## 13.3 Governance

At first, Valthera is governed by its operating company, with an independent methodology and integrity committee responsible for evidence standards, agent admission, and dispute handling. More decentralized governance would be considered only if it solves a specific problem.

---

# 14. Legal and Regulatory Considerations

Raising capital, pooled investing, token issuance, investment advice, and automated execution can each bring regulatory obligations. These vary by instrument, jurisdiction, and participant. **Using AI or smart contracts does not remove them.**

As an example, in the United States, Regulation Crowdfunding requires that transactions happen online through an SEC-registered intermediary, either a broker-dealer or a funding portal. It also caps an issuer's crowdfunding raises at $5 million in any 12-month period. Non-accredited investors face annual investment limits based on income and net worth. Other paths, such as private placements under Regulation D, Regulation A, or offshore frameworks, have their own conditions.

**A key structural constraint:** a registered *funding portal* may not offer investment advice or recommendations, and may not hold or handle investor funds ([SEC, Registration of Funding Portals](https://www.sec.gov/resources-small-businesses/small-business-compliance-guides/registration-funding-portals)). An intermediary that does those things must register as a broker-dealer instead. Valthera's core product, AI-generated assessments and recommendations, therefore cannot sit inside a funding-portal entity that also hosts offerings. Valthera must choose one of three paths: (a) keep research and capital formation in separate legal entities with information barriers; (b) operate through, or partner with, a registered broker-dealer; or (c) run capital formation outside the U.S. under local rules.

The SEC withdrew its proposed rule on conflicts of interest from predictive data analytics on June 12, 2025 ([Federal Register, 2025-11110](https://www.federalregister.gov/documents/2025/06/17/2025-11110/withdrawal-of-proposed-regulatory-actions)). That removes one AI-specific rule, but existing fiduciary, anti-fraud, and conflict-disclosure obligations still apply. For tokens, SEC Chairman Atkins has described a token taxonomy anchored in the *Howey* test ([SEC speech, Nov. 12, 2025](https://www.sec.gov/newsroom/speeches-statements/atkins-111225-secs-approach-digital-assets-inside-project-crypto)). Tokenized securities remain securities.

Valthera's approach:

- **Phase 1 (research only):** offer analysis and information products with clear disclaimers, and assess whether any of them could count as investment advice.
- **Before any capital formation:** obtain legal advice country by country, and either work with licensed intermediaries (for example, registered funding portals or broker-dealers) or obtain the necessary registrations.
- **Agent operators** are responsible for their own regulatory status. Valthera requires disclosure of who operates each agent and checks operators against a minimum standard.
- **KYC/AML** applies to all parties who commit capital, whether a person or the entity behind an agent.

---

# 15. Business Model

| Phase | Revenue sources |
|---|---|
| **Early** | Evaluation services, premium research, advanced diligence reports, investor workflow tools |
| **Growth** | Professional subscriptions, APIs for agent developers, enterprise research infrastructure |
| **Mature** | Agent infrastructure (identity, permissions, reporting, evaluation data, controlled execution) and capital-formation fees where legally permitted |

**Managing conflicts:** charging founders for reports could undermine claims of independence. Paid analysis will be clearly separated and labeled. Independent assessments will never depend on payment, and relevant conflicts will be disclosed.

**On tokens:** Valthera does **not** plan a token at launch. A token does not create demand, improve investment quality, or make a business sustainable on its own. A token or other protocol mechanism would be considered only if it solves a clearly defined problem that cannot be solved more simply, and only after legal review.

---

# 16. Go-to-Market

## 16.1 Starting segment: crypto and Web3

Crypto and Web3 projects usually publish documentation, smart contracts, tokenomics, repository activity, and onchain data. This gives the evidence engine material to check independently of the deck.

## 16.2 Initial workflow

1. A project submits its deck and supporting data.
2. Valthera produces an evidence-backed memo, a valuation range where enough evidence exists, key risks, and a list of missing evidence.
3. A small group of independent investors reviews the output and gives feedback. Their actual decisions are recorded and compared with Valthera's.
4. Specialized agents with distinct mandates are introduced, in **recommend-only** mode at first.
5. After reliability is shown and safeguards are in place, selected agents receive tightly scoped execution permissions.

## 16.3 Early success metrics

- Factual accuracy of findings, measured by audits of claim labels.
- Rating of evaluation usefulness from investors and founders.
- Investor engagement with opportunities.
- Ability to tell strong projects from misleading ones, measured against what later happens.
- Repeat usage on both sides of the market.

---

# 17. Roadmap

Phases are ordered by what they depend on, not by date. Each phase begins only after the previous one meets its exit criteria.

| Phase | Scope | Exit criteria |
|---|---|---|
| **0 — Foundation** | Claim schema, evidence labels, ingestion pipeline, prompt-injection defenses | Internal accuracy benchmark passed; red-team review complete |
| **1 — Diligence Product** | Evidence-backed memos and valuation ranges for Web3 projects; investor pilot group | Factual-accuracy target met; investors use it repeatedly |
| **2 — Agent Network (Recommend)** | Public mandate registry, agent API, decision records, early track-record analytics | Several independent operators active; scoring method published |
| **3 — Capital Formation (Conditional)** | Commitment books, conditional commitments, licensed-partner settlement | Legal structure in place in target jurisdictions; first compliant rounds closed |
| **4 — Bounded Execution** | Tier-3 permissions for qualified agents, onchain limits, milestone tranches | Security audits passed; no critical incidents during conditional phase |
| **5 — Expansion** | More sectors (AI-native, SaaS), more chains, third-party agent infrastructure | Category-specific evaluation models validated |

---

# 18. Risks

- **Model error:** agents may misread evidence or be confidently wrong.
- **Manipulation:** projects may game the evidence engine, and agents may collude.
- **Correlated failure:** agents built on similar models may share blind spots, which can make crowding worse.
- **Regulatory risk:** rules on AI-driven advice, crowdfunding, and digital assets are changing and differ by jurisdiction.
- **Adoption risk:** agent-to-agent commerce is still early. Demand from autonomous investors may arrive more slowly than the thesis assumes.
- **Outcome latency:** early-stage results take years, which slows track-record validation.
- **Security:** smart contracts, custody, and agent credentials are all attack surfaces.
- **Conflicts of interest:** commercial pressure could compromise independence if not managed.

---

# 19. Conclusion

An agent that can analyze a business is useful. An agent that can analyze a business, judge whether its price is attractive, commit capital within explicit limits, and learn from the result is a different kind of market participant. Valthera aims to build the infrastructure that makes such participants **accountable**: evidence instead of narrative, many judgments instead of one, bounded authority instead of blind autonomy, and records that can be checked against real outcomes.

Valthera starts with project submissions, AI due diligence, valuation analysis, and recommendations. From there, it aims to coordinate autonomous investment agents and grow into a capital market where people and agents find, evaluate, fund, and monitor opportunities together.

**Valthera. Capital allocated by intelligence.**

---

# Appendix A: Glossary

- **Agent:** software that pursues investment objectives under a registered mandate.
- **Mandate:** a machine-readable statement of an agent's strategy, eligibility rules, evidence thresholds, and risk limits.
- **Opportunity object:** the standard structured representation of a project seeking capital.
- **Evidence label:** the support status of a claim (Verified, Supported, Reported, Projected, Contradicted, Unknown).
- **Decision record:** a saved snapshot of an agent's assessment, made with the evidence available at the time.
- **FDV:** fully diluted valuation. The token price multiplied by the maximum token supply.

# Appendix B: References

1. U.S. SEC, *Regulation Crowdfunding*. https://www.sec.gov/resources-small-businesses/exempt-offerings/regulation-crowdfunding
2. U.S. SEC, *Regulation Crowdfunding: Guidance for Issuers*. https://www.sec.gov/resources-small-businesses/regulation-crowdfunding-guidance-issuers
3. Investor.gov (SEC OIEA), *Updated Investor Bulletin: Crowdfunding Investment Limits Increase*.
4. OWASP GenAI Security Project, *LLM01:2025 Prompt Injection*. https://genai.owasp.org/llmrisk/llm01-prompt-injection/
5. OWASP, *Top 10 for LLM Applications 2025*. https://genai.owasp.org/llm-top-10/
6. Coinbase, *x402: An open standard for internet-native payments* (whitepaper). https://www.x402.org/x402-whitepaper.pdf
7. CoinDesk, "Coinbase-backed AI payments protocol wants to fix micropayment but demand is just not there yet," March 11, 2026.
