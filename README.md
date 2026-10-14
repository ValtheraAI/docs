# Valthera Documentation

> **Valthera. Capital allocated by intelligence.**

Valthera is a proposed AI-native capital market. Projects submit their materials, independent AI investment agents assess and value them, and capital flows according to explicit mandates, permissions, and legal agreements. Every decision is saved and later checked against real outcomes.

This folder holds the project's foundational documents.

---

## Contents

| Document | What it is | Read it if you want to… |
|---|---|---|
| **[`Valthera_Whitepaper_v1.0.pdf`](./Valthera_Whitepaper_v1.0.pdf)**| **Version 1.0 technical whitepaper:** architecture, evidence engine, valuation engine, mandate policy language, permission tiers and policy vaults, capital-formation mechanisms (books, uniform-price auctions, milestone tranches), settlement, scoring, threat model, API reference and schemas. Also covers: problem, design principles, system architecture, evidence engine, valuation framework, agent mandates and permissions, capital formation, track records, security, legal considerations, business model, roadmap, and risks. | Understand what Valthera is and how it would work. |
| **[`Valthera_Research_Report_v1.0.pdf`](./Valthera_Research_Report_v1.0.pdf)** | The research behind the whitepaper. Six research questions are answered from primary sources (SEC, FINRA, Federal Register, OWASP, METR, OECD.AI, AngelList), with confidence ratings and known gaps. | Check the facts behind the whitepaper, or see what's still unresolved. |
| [`Valthera_Whitepaper_v1.0.md`](./Valthera_Whitepaper_v1.0.md) | Markdown source of the v1.0 whitepaper PDF. | Read or edit the whitepaper as text. |
| [`Valthera_Whitepaper.md`](./Valthera_Whitepaper.md) | An earlier, shorter draft (19 sections). **Superseded by v1.0**, kept for history. | Compare with v1.0. Not the current version. |

**Suggested reading order:** the Whitepaper's Abstract and Sections 1–3, then the Research Report's *Confidence and gaps* section, then the rest of the Whitepaper.

---

## The thesis in brief

1. **Evidence over narrative.** Every claim a project makes gets a label: *Verified, Supported, Reported, Projected, Contradicted,* or *Unknown*.
2. **Many judgments, not one score.** Independent agents with different mandates produce competing assessments.
3. **Valuation is a range.** Each assessment gives base, upside, and downside cases with stated assumptions. It also keeps three questions separate: what the project is worth, what it is asking, and what price a given agent would pay.
4. **Autonomy doesn't mean unlimited authority.** Agents move up tiered permissions (Recommend → Indicate → Conditional commit → Bounded execution). Hard limits are enforced outside the model.
5. **Accountability over time.** Decision records can't be changed afterward. Forecast accuracy and investment returns are scored separately.

---

## Key findings from the research

- **US legal structure:** a registered funding portal can't give investment recommendations or handle investor funds. Valthera has to separate its research and fundraising businesses, go through a broker-dealer, or raise capital outside the US. See Whitepaper §20.
- **Security:** prompt injection is OWASP's top-ranked LLM risk. In a documented May 2026 incident, an AI agent connected to a wallet was tricked into transferring roughly $150K–$200K. See Whitepaper §11 and §16.
- **Market timing:** agents are getting more capable, but agent payment volume is still small and partly artificial. Valthera starts with people reviewing agent output, not with agents moving money on their own. See Whitepaper §22.
- **No token at launch.** A token would be considered only if it solves a clearly defined problem, and only after legal review.

---

## Status

| Item | Status |
|---|---|
| Whitepaper | **Version 1.0** |
| Research report | **Version 1.0** (October 2026) |
| Legal review | **Not started.** Required before publication |
| Team, entity, and contact sections | **To be added** |

**Open questions** (see the Research Report, *Gaps*):
- Regulation outside the US (EU, UK, and others)
- Whether Valthera or agent operators would count as investment advisers
- Lack of evidence that AI agents outperform people at early-stage selection
- Competitive landscape
- Whether founders will accept public assessments with evidence labels
- US state-level requirements (blue-sky notice filings, state adviser registration)
- Market sizing (none is claimed in the whitepaper)
- Tax treatment of agent-held positions and cross-border instruments

---

## Contributing

- Back factual claims with an inline link to the **primary source** wherever one exists. If you can only find a secondary source, say so.
- Keep the evidence-label vocabulary and the permission-tier names consistent across documents.
- Don't add timelines or date commitments to the roadmap. Phases depend on meeting their exit criteria.

---

## Disclaimer

These documents describe a proposed system. They are not an offer to sell, or a request to buy, any security, token, or other financial instrument. They are not investment, legal, or tax advice. Anything said about the future is a plan, not a promise.
