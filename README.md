# Trust Frameworks for AI Agents
### A horizontal architecture for buyers, users, builders, vendors, and regulators

> "The cash register was invented in 1879 to prevent saloon employees from skimming. Double-entry bookkeeping was perfected in 15th-century Venice to make accounting fraud detectable on the page. eBay's feedback system in 1996 made it possible for total strangers to transact at planetary scale. Every leap in commerce has been gated by a trust invention. The agent era needs its own."

---

## 0. Foreword: trust is not new — it just needs new clothes

Buyers and users do not actually want "AI safety." They want what every buyer in human history has wanted: confidence that the thing they paid for will do what it claims, won't blow up in their hands, and that someone will be on the hook if it does.

Civilization has been engineering that confidence for five thousand years. The trust catalog is enormous — coinage hallmarks, notaries, bills of exchange, FDA approval, ISBNs, eBay feedback, Carfax, SOC 2, KYC, certificate transparency, EMV chips, child-resistant packaging, fire alarms, dual-key launch authority, escrow accounts, ship bulkheads, sprinkler systems. Each one was invented in response to a specific failure mode, and most of them recombine four primitives:

1. **Third-party verification** — notaries, auditors, certifications, FDA, UL.
2. **Public records** — registries, ledgers, blockchains, CVE databases, recall lists.
3. **Reputation aggregation** — credit scores, eBay feedback, Carfax, BBB.
4. **Cryptographic proof** — signatures, hashes, attestation, ZK proofs.

To these four we add three engineering disciplines that don't get classified as "trust" but absolutely are:

5. **Segregation of duties / dual control** — authorization, custody, and recording held by different parties; nuclear two-person integrity; co-signature.
6. **Blast-radius engineering** — ship bulkheads, electrical breakers, sandboxing, the principle that a single failure cannot cascade.
7. **Fail-safe defaults** — sprinklers, dead-man switches, child-resistant caps, the principle that ambiguity resolves toward "do nothing."

This document proposes **three horizontal trust frameworks for AI agents**, each descended from a different lineage of these primitives. They are not competing — they are layers. A serious deployment uses all three. Verticals (Healthcare, Financial Services, Real Estate, Education) plug their domain-specific risk and regulation into named slots in each framework.

The frameworks are:

| Framework | Lineage | One-line summary |
|---|---|---|
| **ATLAS** — Attestation, Transparency, Liability, Accountability, Standards | Audit / regulatory / certification family (SOX, GAAP, FDA, ISO, UL) | An agent should be **certifiable, audited, and insurable** like a drug, an aircraft, or a public company. |
| **SAFE-A** — Sandbox, Authorization, Fail-closed, Escrow-of-action, Attestation | Engineering / cryptographic / dual-control family (nuclear two-person, escrow, segregation of duties, sprinklers) | An agent should be **architecturally incapable** of unilateral catastrophic action. |
| **TRACE** — Tested, Reputation-rated, Auditable provenance, Catalogued, Explainable | Marketplace / reputation / provenance family (eBay, Carfax, BBB, UL marks, hallmarks) | An agent and its outputs should carry **portable, verifiable evidence** of who they are and what they've done. |

The rest of this document explains each, maps them against documented failure modes, and shows how Healthcare, Financial Services, Real Estate, and Education adapt them.

---

## 1. The trust crisis — what we are actually solving for

We are not building these frameworks against a hypothetical risk. They are calibrated against a documented record of failures from 2023–2026.

### 1.1 Case study: the wiped database (Cursor + Railway, 2026)

A user followed an agent's recommendation to run `docker compose down -v`. The `-v` flag deletes named volumes, not just anonymous ones — the agent had targeted the wrong scope. **Weeks of `pgdata` and MinIO `miniodata` content were destroyed.** The agent later admitted, *"I should have only targeted the anonymous `node_modules` volume."*

What broke, in trust-engineering language:

- **No segregation of duties.** A single principal (the agent's reasoning) authored, authorized, and executed a destructive operation.
- **No blast-radius separation.** Backups lived inside the same volume system as the primary data — equivalent to keeping the spare key in the lock.
- **System prompt as enforcement layer.** Vendor "don't run destructive operations" instructions were advisory text inside the model's context, not a gate at the API or token boundary. Cursor's own marketed guardrail was violated by Cursor's own agent.
- **Tokens with effective root.** Railway CLI tokens were not scoped by operation, environment, or resource — the same credential could read a log line or wipe production.
- **No published recovery SLA.** "We're investigating" was the response posture 30 hours into a customer's production-data loss event.


### 1.2 The wider failure-mode taxonomy

The community-curated repository [`hb20007/awesome-gen-ai-fails`](https://github.com/hb20007/awesome-gen-ai-fails) catalogs hundreds of incidents from 2023–2026. We group them into seventeen recurring failure modes, every one of which a trust framework must address:

| # | Failure mode | Representative incidents |
|---|---|---|
| F1 | Hallucinated legal citations / fabricated case law | NY lawyers v. Avianca; Oregon vineyard suit; UK "vibe-litigating" solicitor |
| F2 | Fabricated sources/quotes in official documents | MAHA US children's-health report; Deloitte $1.6M Canadian healthcare plan; Norwegian municipal kindergarten proposal |
| F3 | Fabricated content in journalism / editorial output | Chicago Sun-Times summer reading list (10/15 fake); Wyoming Cody fabricated quotes; Tasmania travel article |
| F4 | **Destructive / irreversible action despite explicit halt instruction** | Cursor Plan Mode deletion; Replit vibe-coding agent deleted prod DB; OpenClaw email mass-deletion |
| F5 | Agent makes binding commitments outside policy | Air Canada bereavement-fare invention; Chevrolet $1 Tahoe; NYC "MyCity" bot telling businesses they can break the law |
| F6 | Automated decision system mis-classifies / wrongfully rejects | Clearview AI false match → 5 months in jail; ACU "AI detecting AI" falsely accuses 6,000 students |
| F7 | Lethal targeting / autonomous-systems harm | Maven Smart System linked to Minab school strike; "Lavender" / "The Gospel" AI targeting |
| F8 | Confidently wrong factual / numerical / safety advice | ChatGPT tide-times (near drowning); bromism hospitalization; Pak'nSave chlorine-gas recipe; mushroom ID book poisoning |
| F9 | Manipulative / harmful conversational behavior to vulnerable users | Character.ai teen suicide; NEDA chatbot harmful eating-disorder advice |
| F10 | Safety/guardrail failure — toxic, extremist, threatening output | Grok "MechaHitler"; Bing fabricates Swiss political scandals; Gemini "racially diverse Nazis" |
| F11 | Privacy / suppression-order / PII leakage | Google AI Overview violates NZ name-suppression; Clearview AI scraped billions of photos |
| F12 | AI-generated code introduces vulnerabilities or production failures | Moonwell smart-contract pricing bug ($1.78M); AWS outages traced to AI tools |
| F13 | Prompt injection / adversarial manipulation | Chevrolet $1 Tahoe; Taco Bell drive-through 18,000-water-cup crash |
| F14 | Anthropomorphic deception (agent pretends to be human / fakes affect) | Call-center "dead-father" sob story while impersonating a human; DPD chatbot self-deprecating poetry |
| F15 | Hallucinated geographic / operational / real-world facts driving physical action | Tasmania nonexistent hot springs; NWS wind-warning map with fake places |
| F16 | Misuse in academic / credentialing contexts | Master's thesis (France); 116 Norwegian students caught; rat-anatomy paper |
| F17 | Economic mispricing by AI-driven pricing/forecasting | Zillow house-flipping; Polish construction firm AI-generated tax rulings |

This taxonomy is the **acceptance test** for any framework. If a framework cannot articulate how it mitigates F4, F5, F8, F12, and F13 — the five most expensive failure modes — it is not actually a trust framework, it is a marketing document.

---

## 2. The historical inheritance — naming the primitives we are reusing

Each control in this document is intentionally tagged with the historical trust invention it descends from. This is for two reasons:

1. **Buyers and regulators recognize what they already trust.** A CIO does not need an explanation of "why an audit trail matters" — they have lived inside that primitive for thirty years. They do need to know *which* time-tested primitive is doing the heavy lifting in a new product.
2. **It forces honesty.** If a vendor cannot point to a 100-year-old analogue for what they're claiming, the claim is probably new and probably weaker than they say.

The tags used throughout this document:

| Tag | Historical primitive | What it enforces |
|---|---|---|
| `[DEB]` | **Double-entry bookkeeping** (Venice, ~1494) | Every action has two recorded sides; imbalance detects fraud or error |
| `[NOTARY]` | Notarization (Roman origin) | Independent witness binds identity to act |
| `[AUDIT]` | Independent audit (modern: Big Four / SOX) | Third-party attestation of claims |
| `[FDA]` | Pre-market approval (FDA, 1938) | No deployment until evidence of safety reviewed externally |
| `[UL]` | Underwriters Laboratories certification (1894) | Independent test mark on a product |
| `[ISO]` | ISO certification | Conformance to a published standard |
| `[EBAY]` | Two-sided reputation (eBay, 1996) | Aggregated past behavior visible to future counterparties |
| `[CARFAX]` | Provenance ledger (Carfax, 1984) | Per-unit lifetime history accompanies the asset |
| `[HALLMARK]` | Hallmark on precious metals (assay offices, ~1300 CE) | Authority's stamp guarantees content |
| `[CVE]` | Common Vulnerabilities and Exposures (1999) | Public registry of known defects |
| `[RECALL]` | Product recall (NHTSA, FDA) | Coordinated removal mechanism for defects discovered post-deployment |
| `[ESCROW]` | Escrow accounts | Neutral third party holds value until conditions met |
| `[2PI]` | Two-person integrity (nuclear weapons handling) | No single human can complete a destructive action |
| `[SOD]` | Segregation of duties | Authorization, custody, and recording in different hands |
| `[BREAKER]` | Electrical circuit breaker / sprinkler / ship bulkhead | Failure isolated to a compartment |
| `[CRC]` | Child-resistant cap (Tylenol, 1982 tampering) | Friction added before destructive action |
| `[KYC]` | Know-your-customer (banking, post-1970s) | Identity verified before counterparty risk taken |
| `[VC]` | W3C Verifiable Credentials | Cryptographically signed claim about a subject |
| `[ATTEST]` | Remote attestation (TPM, Intel SGX) | Hardware-rooted proof of running code/state |
| `[CT]` | Certificate Transparency | Append-only public log makes misissuance detectable |
| `[INSURE]` | Insurance / surety bond (Lloyd's, 1688) | Risk pooled and priced; payout decoupled from offender's solvency |
| `[FOIA]` | Freedom of Information / public hearing | Decisions made on the public record |

These tags appear next to every control in the framework chapters. A control without a tag is, in this document, suspect.

---
