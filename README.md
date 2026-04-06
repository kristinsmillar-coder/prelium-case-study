# Building an Autonomous AI Analysis Engine for Data-Intensive Industries

**A Technical Case Study from Prelium**

Pierce Millar, Jackson [Redacted], Keegan Tingey
April 2026

---

## Abstract

Over a six-month period, a three-person team built an autonomous AI system capable of processing 13,650 commercial real estate investment opportunities in 36 hours — work that would take a team of analysts several months. The system extracts structured data from unstructured documents, cross-references it against 50+ external data sources, scores opportunities against configurable investment criteria, and learns from every human decision to improve over time. Total infrastructure cost: $48/month. Total AI cost for 197 verified deals: $0.83. This paper describes the architecture, methodology, and results — and explores the broader implications for any industry built on analyzing large volumes of complex documents.

---

## 1. The Problem

Commercial real estate analysts spend 4-6 hours per deal manually reviewing offering memoranda (OMs) — dense PDF documents ranging from 20 to 200+ pages containing financial data, lease terms, property specifications, market context, and tenant information. A typical mid-market fund receives 50-100 of these per week from broker networks. The analyst's job is to extract roughly 20 key data points from each document, enter them into an Excel model, cross-reference them against public records, score the deal against the firm's investment criteria, and present a recommendation.

The bottleneck is not intelligence — it is throughput. Senior analysts know within minutes whether a deal is interesting. But the mechanical process of extracting, entering, verifying, and formatting data consumes the vast majority of their time. This is a pattern that exists across many industries: insurance underwriting, legal due diligence, medical record review, credit analysis, and regulatory compliance all share the same fundamental structure — high-volume document intake, structured data extraction, multi-source verification, and human decision-making on the output.

## 2. What We Built

### 2.1 The Extraction Engine

The core of the system is a three-pass extraction pipeline we call BRAID (Bi-directional Retrieval and Iterative Deconfliction):

**Pass 1 — Extraction.** A large language model (Claude Sonnet) reads the full document and extracts 20 structured fields: property name, address, asking price, capitalization rate, net operating income, square footage, occupancy, tenant information, lease terms, and more. Each field is returned with a confidence score (0-1).

**Pass 2 — Verification.** The same document is re-read with a different prompt focused on catching errors. The model is specifically instructed to verify that every extracted number actually appears in the source document. Any field where the Pass 1 and Pass 2 values disagree is flagged.

**Pass 3 — Gap-filling.** Missing fields are derived from available data where mathematically possible (e.g., if price and cap rate are known, NOI can be calculated). Remaining gaps are marked as absent rather than estimated.

The result is a structured JSON object with 20 fields, each tagged with a confidence score and source page reference. The extraction cost is $0.004 per document.

### 2.2 The Eight-Agent Architecture

After initial extraction, deals that score above a configurable threshold enter a deep analysis pipeline powered by eight specialist AI agents, each with a distinct domain:

| Agent | Domain | Example Output |
|-------|--------|----------------|
| **Scout** | Market demographics, supply pipeline, population trends | "Tampa industrial vacancy is 4.2%, down 80bps YoY. 1.2M SF under construction within 5 miles." |
| **Spectre** | Ownership intelligence, entity research, property history | "Owner is a 2019 LLC registered to a family office. Property last traded in 2016 at $6.8M." |
| **Sentinel** | Environmental risk, flood zones, regulatory exposure | "Property is in FEMA Zone X (minimal flood risk). No EPA facilities within 1 mile." |
| **Atlas** | Financial underwriting, comparable sales, rent analysis | "Comparable industrial sales in the submarket averaged $215/SF over the past 12 months." |
| **Arbiter** | Legal risk, liens, litigation, tenant credit | "Anchor tenant (FedEx) has investment-grade credit. No UCC filings against the owner entity." |
| **Vault** | Macro context, interest rates, capital markets | "10-year Treasury at 4.32%. Industrial cap rates nationally compressed 15bps this quarter." |
| **Forge** | Physical condition, construction costs, capital expenditure | "28' clear height is market-standard for modern logistics. Roof replacement estimate: $4.50/SF." |
| **Prism** | Synthesis and citation verification | Cross-references all seven agents' outputs against source documents. Any unsupported claim is flagged or removed. |

Each agent has access to a curated set of data APIs relevant to its domain. In total, the system connects to 50+ external data sources including the U.S. Census Bureau, Bureau of Labor Statistics, FEMA, EPA, SEC EDGAR, FRED (Federal Reserve Economic Data), HUD, county property appraiser and clerk of court systems across multiple states, and commercial real estate listing platforms.

The agents operate in four sequential phases, with agents within each phase running concurrently. After every pipeline run, each agent is automatically graded (A+ through F) on completeness, accuracy, and citation quality. Any agent scoring below A- is diagnosed, has its instructions adjusted, and is re-run — without human intervention.

### 2.3 The Quality Harness

Perhaps the most important part of the system is not what it produces, but how it verifies what it produces. We built a seven-layer quality harness through which every output must pass:

**Layer 1 — Unit Tests (155 tests).** Individual function-level verification. Does the parser handle edge cases? Does the scorer produce consistent results?

**Layer 2 — Integration Tests.** End-to-end flow for each of seven property types (industrial, office, retail, multifamily, self-storage, NNN, mixed-use). A failure in any property type blocks deployment.

**Layer 3 — Regression Tests.** Every output is compared against 10 saved baseline results. Any field that drifts more than 15% from baseline is flagged.

**Layer 4 — Simulation Battery.** Stress tests: 20 deals sustained throughput, 15 concurrent extractions, malicious input handling, Unicode edge cases, SQL injection attempts, multi-tenant isolation, and crash recovery. 29 of 31 simulations passing.

**Layer 5 — Grade Pipeline (5 sub-layers).** Automated quality scoring:
- Completeness: Are all 20 fields present with confidence > 0.8?
- Sanity: Is the cap rate between 1-30%? Is the price positive? Is the year built between 1800 and next year?
- Cross-reference: Does NOI approximately equal price times cap rate? Does price per square foot approximately equal price divided by square footage?
- Regression: Does the deal score fall within 2 points of baseline?
- Audit: Can the score be independently recalculated and match?

**Layer 6 — Visual QA.** The system opens the actual output files (populated Excel spreadsheets, HTML reports) using computer vision, takes screenshots, and cross-references visible values against the source document. This catches formatting errors, misaligned cells, and display bugs that code-level tests miss.

**Layer 7 — Adversarial Tests (13/13 passing).** Password-protected PDFs, scanned images, Unicode bombs, 500-page documents, empty files, wrong formats, injection attempts, and deliberately misleading data.

The philosophy behind this harness comes from a principle articulated by Phil Bogdanoff at OpenAI: "It's a skill issue if the model doesn't produce accurate output. Your job is to build the right harness, the right prompts, the right eval." The model is capable. The engineering challenge is reliability.

### 2.4 The Learning Loop

The system improves with every human decision. When an analyst reviews a deal and agrees with the system's recommendation, that confirmation reinforces the scoring weights. When the analyst disagrees — overriding the system's GO with a NO-GO, or vice versa — the override is captured with context: which fields were checked, why the disagreement occurred, and what the correct decision should have been.

After accumulating sufficient decisions (10+), the system analyzes override patterns: which criteria are most frequently overridden, for which property types, in which markets. It proposes weight adjustments, back-tests them against all historical decisions, and commits the new weights only if the override rate decreases.

The trajectory we project based on simulation:
- **Month 1:** 40% override rate. The system is learning.
- **Month 3:** 15% override rate. The system knows preferences.
- **Month 6:** 5% override rate. The system predicts decisions.
- **Month 12:** 2% override rate. The analyst's judgment is encoded.

This creates a compounding advantage. After 12 months, a client switching to a competitor would need to retrain the system from scratch — losing a year of accumulated institutional knowledge. With five or more clients, anonymized cross-client intelligence begins to emerge: "Deals with this profile have a 73% close rate across all our clients" or "Asking price is 12% above what similar deals traded for this quarter."

## 3. How We Built It

### 3.1 Infrastructure

The entire system runs on a single DigitalOcean VPS ($48/month): Ubuntu 24.04, 8GB RAM, 4 vCPUs. A Mac Mini M4 ($599) serves as a secondary node for browser automation tasks requiring residential IP addresses (county record systems block datacenter IPs). PostgreSQL handles the database. n8n (open-source) handles workflow orchestration. PM2 manages processes.

Total infrastructure cost: under $100/month.

### 3.2 Model Economics

We use a tiered model strategy to keep AI costs negligible:

| Task | Model | Cost |
|------|-------|------|
| Email classification and routing | Claude Haiku | ~$0.01/email |
| Document extraction and scoring | Claude Sonnet | ~$0.004/document |
| Complex underwriting synthesis | Claude Opus | ~$2-5/analysis |

Prompt caching reduces costs by approximately 90% for repeated schema patterns. The total AI cost for processing 197 deals through extraction was $0.83.

For deep pipeline analysis (all eight agents), the cost is approximately $3-8 per deal depending on document complexity and the number of external data sources consulted. Even at the high end, this represents less than 0.001% of a typical deal's asking price.

### 3.3 The Build Process

The system was built using what we call the Factory pattern, inspired by Anthropic's own internal development methodology:

1. **Planner** reads the specification and identifies the next quality gate to turn from RED to GREEN.
2. **Builder** creates a feature branch and writes code according to the spec.
3. **Evaluator** (a separate AI agent with fresh context) reviews the code, runs all tests, and either passes or rejects with specific feedback.
4. **Cross-model review** uses a different AI model family (OpenAI Codex) to catch biases specific to any single model.

The builder never evaluates its own work. This separation is critical — when you self-evaluate, you rationalize your own bugs because you understand your intent. An evaluator with fresh context only sees the code, not the intent.

The current codebase is 4,244 lines of Python across the core pipeline, with an additional 130+ tool scripts for data API integrations, browser automation, and quality assurance.

## 4. Results

### 4.1 Throughput

The system processed 13,650 commercial real estate properties in 36 hours during an initial batch run. This included downloading offering memoranda, extracting structured data, scoring against investment criteria, and generating summary reports. Equivalent analyst work at 4 hours per deal: approximately 54,600 hours, or 26 analyst-years.

### 4.2 Accuracy

Across 10 regression baselines (6 real offering memoranda from actual broker submissions + 4 synthetic edge cases), the extraction engine achieves:
- 20/20 fields extracted on well-formatted documents
- Confidence scores above 0.8 on 18/20 fields (the two common low-confidence fields are WALT and clear height, which are inconsistently reported across OMs)
- Cross-reference verification catches discrepancies between stated NOI and calculated NOI (price x cap rate) at a 10% tolerance

### 4.3 Cost

| Component | Monthly Cost |
|-----------|-------------|
| VPS (8GB/4vCPU) | $48 |
| Mac Mini (amortized) | ~$25 |
| External data APIs | ~$15-30 |
| AI model usage (per 100 deals) | ~$0.40 extraction / ~$500 deep analysis |
| **Total (100 deals/month)** | **~$120-600** |

For context, a junior CRE analyst costs $65,000-85,000/year plus benefits. The system processes more deals, faster, at approximately 1% of the cost — and improves with every decision.

## 5. Implications Beyond Commercial Real Estate

The architecture described here is not CRE-specific. The pattern — document intake, structured extraction, multi-source enrichment, configurable scoring, human-in-the-loop learning — applies to any domain where professionals spend significant time manually processing documents and making structured decisions.

### 5.1 Life Settlements and Insurance

Consider a firm that acquires life insurance policies on the secondary market. The underwriting process involves reviewing medical records — sometimes thousands of pages per individual — to forecast mortality. The firm collects updated medical records annually and re-evaluates each policy.

This is structurally identical to CRE deal analysis:
- **Document intake:** Medical records (hundreds to 20,000 pages) replace offering memoranda
- **Extraction:** Diagnoses, medications, lab results, vital trends replace property financials
- **Enrichment:** Actuarial tables, clinical research, pharmaceutical databases replace market data
- **Scoring:** Mortality probability estimation replaces investment scoring
- **Learning loop:** Actual mortality outcomes (known, since policies are held to maturity) provide ground truth feedback — even stronger signal than CRE override data

A life settlement firm with 20 years of historical data — policies purchased, mortality predictions made, actual outcomes observed — possesses an extraordinarily valuable training dataset. Every prediction that was wrong teaches the system where its models fail. Every prediction that was right reinforces what works.

The privacy constraint (HIPAA) is addressed through local deployment. The AI models and all patient data reside on hardware physically within the firm's control. No data transits public cloud infrastructure. The system operates as an additional input to the underwriting process — a virtual panel of specialists that reviews every file — rather than a replacement for human judgment.

### 5.2 The General Pattern

Any industry that matches the following criteria is a candidate for this approach:

1. **High-volume document intake** — receiving more documents than humans can thoroughly review
2. **Structured decision-making** — the decision can be decomposed into measurable criteria
3. **External data enrichment** — public or licensed data sources can supplement the primary documents
4. **Historical outcomes** — past decisions can be evaluated against actual results
5. **Privacy requirements** — sensitive data necessitates controlled infrastructure

Examples include: credit underwriting, legal discovery, regulatory compliance review, pharmaceutical clinical trial analysis, and insurance claims processing.

## 6. What We Learned

**The harness matters more than the model.** Large language models are capable of extraordinary analysis. The engineering challenge is not getting them to produce good output once — it is getting them to produce reliable output every time. The seven-layer quality harness we built catches errors that no single test would find. The adversarial suite catches errors that no honest test would simulate.

**Cost is not the barrier.** At $0.004 per document extraction, AI model costs are negligible compared to the value of the decisions they inform. The real cost is building the infrastructure to make the output trustworthy.

**The moat is in the learning loop.** Technology can be replicated. What cannot be replicated is 12 months of a specific firm's decision patterns, encoded into scoring weights that predict their preferences. The system becomes more valuable — and harder to replace — with every decision it processes.

**Privacy-first architecture enables adoption.** Firms handling sensitive data (medical records, financial information, legal documents) will not upload that data to third-party cloud services. Local deployment on controlled hardware — a Mac Mini in a closet — eliminates this objection entirely.

## 7. About Prelium

Prelium is an AI consulting firm that builds autonomous analysis systems for data-intensive businesses. We go into a firm, study their workflow, and wire AI into their existing tools. The client never sees our software — they see their Excel auto-populate, their inbox pre-sorted, their reports pre-written.

Our team:
- **Pierce Millar** — COO. Business development, client relationships, workflow analysis.
- **Jackson [Redacted]** — CEO. System architecture, AI engineering, pipeline development.
- **Keegan Tingey** — CMO. Growth strategy, market positioning, operations.

Contact: hello@prelium.ai | prelium.ai

---

*This document was prepared for informational purposes. Technical details have been simplified for accessibility. The system described is in active development with production deployments underway. All statistics cited are from verified internal records as of April 2026.*
