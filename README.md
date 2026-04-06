# Axon: A Nine-Agent Autonomous AI System That Replaced 26 Analyst-Years of Work in 36 Hours

**How Two People Built an AI Deal-Sourcing Engine for Commercial Real Estate**

Pierce Millar & Jackson Kiil
April 2026

---

## Abstract

Over six months, a two-person team built Axon — a nine-agent autonomous AI system that processes commercial real estate investment opportunities at scale. In its first major batch run, Axon analyzed 13,650 properties in 36 hours, scoring each against configurable investment criteria, pulling data from 130+ external tools and APIs, tracing property ownership chains, detecting financial distress signals, and generating institutional-quality analysis reports. The equivalent analyst workload: approximately 54,600 hours, or 26 years of full-time work. The system runs on a $48/month cloud server. Total AI cost for the first 197 deep-verified deals: $0.83.

This paper describes what we built, how it works, and what it means for any industry that runs on analyzing large volumes of complex documents.

---

## 1. How This Started

We were hired by a family office to source commercial real estate deals. The task was simple: find undervalued properties, analyze them, and bring the best ones to the investment committee.

The problem was also simple: we had no connections. We were working through LoopNet and Craigslist — platforms that every private equity fund on the planet had already picked clean. Three weeks in, we looked at each other and said, "We are never going to find anything this way."

So we asked a different question: what if we could analyze every single deal on the market — not one at a time, but all of them at once? What if we could download thousands of properties and have an AI system score all of them overnight, flag the ones worth pursuing, and tell us exactly why?

That question became Axon.

## 2. What We Built

### 2.1 The Dashboard

The front end is a full web application — React 19, dark Bloomberg-terminal aesthetic, 68 API routes serving live data from a PostgreSQL database holding 12,360+ deals.

**The deal funnel:**

| Stage | Deals | What Happens |
|-------|-------|-------------|
| Intake | 12,360+ | Every property enters the system and gets scored |
| Report Cards | 249 | Scored above threshold — gets a one-page summary |
| Deep Dives | 82 | Full 9-agent analysis with external data enrichment |
| Active Pipeline | 20 | Actively being pursued — LOIs, broker contact, site visits |

The centerpiece is the **Deal Detail page** — a tabbed interface where you can see everything about a property in one place: overview and photos, financials and underwriting, all agent analysis documents, environmental and demographic data, and a deal room with notes, tasks, and activity history.

Other views include a filterable deal table, a pipeline kanban board, an interactive map with every property plotted, a side-by-side deal comparison tool, a broker CRM, market analytics dashboards, and a lease calendar.

### 2.2 The Nine Agents

Axon is not one AI. It is nine specialized agents, each with a distinct role, its own set of tools, and its own instructions. They run in a four-phase pipeline — gathering raw data in parallel first, then layering analysis on top, then synthesizing everything into a final verdict.

**Phase 1 — Intelligence Gathering (parallel)**

| Agent | Role | What It Does |
|-------|------|-------------|
| **Scout** | Head of Market Intelligence | Pulls demographics, employment trends, vacancy rates, comparable sales, construction permits, supply pipeline, and satellite/street-level imagery for the property's submarket. Sources: U.S. Census, Bureau of Labor Statistics, HUD, Redfin, Google Street View, permit databases. |
| **Spectre** | Head of Ownership Intelligence | Traces the property's ownership chain — from the deed, to the registered entity, to the officers and members behind it. Runs background checks. Detects distress signals: late taxes, liens, foreclosure filings, UCC filings, bankruptcy, estate proceedings. Sources: county appraiser/clerk systems across Florida and California, state corporate registries, Enformion, SEC EDGAR. |
| **Sentinel** | Environmental & Risk Assessment | Checks flood zones, EPA facilities, wetlands, natural disaster history, and environmental contamination risk within a defined radius. Sources: FEMA, EPA, National Wetlands Inventory, OpenFEMA. |

**Phase 2a — Financial Analysis (parallel, using Phase 1 data)**

| Agent | Role | What It Does |
|-------|------|-------------|
| **Atlas** | Head of Underwriting | Builds three-scenario financial models (base, upside, downside). Runs 10-year cash flow projections, sensitivity analysis on cap rate and occupancy, return waterfalls, and comparable sales analysis. Sources: Redfin, SEC EDGAR cap rates, FRED economics, insurance estimator, proforma engine. |
| **Arbiter** | Legal & Compliance | Reviews zoning compliance, checks for liens and litigation (lis pendens), reviews tenant credit via SEC filings, and builds a legal risk matrix. Sources: city zoning databases, county clerk records, UCC search, property tax appeals. |

**Phase 2b — Strategy (parallel, using Phase 1 + 2a data)**

| Agent | Role | What It Does |
|-------|------|-------------|
| **Vault** | Capital Markets | Analyzes financing options, sizes debt based on current rates, calculates DSCR sensitivity, matches to potential lenders. Sources: FRED (Treasury yields, SOFR, mortgage rates), FDIC bank data. |
| **Forge** | Asset Management | Develops value-add strategy, estimates capital expenditure budgets, assesses construction costs, evaluates operational improvements. Sources: BLS construction labor costs, NREL utility rates, construction document analysis. |

**Phase 3 — Synthesis**

| Agent | Role | What It Does |
|-------|------|-------------|
| **Prism** | Quality Control & Synthesis | Cross-references every finding from all seven agents against source documents. Any claim that cannot be traced back to a source is flagged or removed. Assembles the master analysis report. |

The ninth agent is **Axon itself** — the orchestrator. It delegates work to the specialists, monitors their progress, grades their output (A+ through F), and automatically re-runs any agent that scores below A-. It operates 24/7, checking in every 30 minutes, scanning for new deals, and communicating results through Discord.

### 2.3 The 130+ Tools

Each agent has access to a curated set of data APIs and automation tools. The full inventory:

**Government & Public Data (25+)**
- Census Bureau (demographics, population estimates, housing)
- Bureau of Labor Statistics (employment, construction labor costs)
- HUD (fair market rents, USPS vacancy rates)
- IRS (county-to-county migration flows)
- FEMA (flood zones, disaster history)
- EPA (environmental facilities, contamination)
- National Wetlands Inventory
- FRED / Federal Reserve (Treasury yields, SOFR, mortgage rates, House Price Index)
- FDIC (bank branch data)
- SEC EDGAR (tenant credit, cap rates, debt filings)
- NREL (utility rates)
- DOT traffic counts (Florida, California)

**County Record Systems (15+ browser automation presets)**
- Hillsborough, Duval, Orange, Sarasota, Manatee County property appraisers (Florida)
- Sacramento, Los Angeles County GIS (California)
- County clerk of court systems (liens, lis pendens, foreclosures)
- County tax collector systems (delinquency detection)
- Sunbiz (Florida corporate registry)
- California Secretary of State entity search

**Commercial Real Estate**
- Redfin (comparable sales)
- Warehouse Exchange (industrial listings)
- Permit aggregators (construction pipeline)
- Rent comp extraction
- Insurance cost estimation
- Construction cost databases
- Zoning lookup (Tampa, Jacksonville, Orlando, Sacramento)

**People & Entity Intelligence**
- Enformion ($0.35/search) — full background: name, age, 37+ addresses, phones, emails, relatives, associates, business records
- Entity chain tracing (LLC → officers → parent entities)
- Google Street View and aerial imagery

**Analysis & Output**
- BRAID 3-pass document extraction (20 structured fields per OM at $0.004/deal)
- Deal scoring engine (configurable 0-100 point system)
- Proforma engine (3-scenario modeling)
- A.CRE Bridge (Excel template population)
- PDF report generation (institutional-quality memos)
- Email categorization and routing

### 2.4 The Scoring Engine

Every property that enters the system gets scored on a 0-100 scale. The criteria are fully configurable per client — different firms care about different things.

**Quick Screen (no API calls, instant):**
- Cap rate must exceed the risk-free rate (10-year Treasury, pulled live from FRED) plus a 200-basis-point spread
- Property type must be in the client's target set

**Full Scoring (0-100 points):**

| Category | Points | What It Measures |
|----------|--------|-----------------|
| Location | 0-25 | Market quality, submarket fundamentals, demographic trends |
| Property Type | 0-25 | Alignment with client's investment thesis |
| Size & Quality | 0-20 | Square footage sweet spot, building class, star rating |
| Value-Add Signals | 0-20 | Below-market rents, vacancy in strong markets, repositioning potential |
| Risk Factors | 0 to -10 | Flood zones, construction risk, regulatory issues |
| Bonus | 0-10 | Pricing below submarket median, confirmed asking price, target size range |

**Disposition tiers:**
- **Pursue** (85+): Immediate action — contact broker, request OM, begin underwriting
- **Priority** (70-84): Strong candidate — schedule for deep dive
- **Review** (50-69): Worth monitoring — add to watchlist
- **Monitor** (30-49): Below threshold but track for changes
- **Skip** (<30): Does not meet criteria

### 2.5 Distress Detection

One of Axon's most valuable capabilities is finding motivated sellers before anyone else knows they're motivated. Spectre maintains a **Distress Signal Matrix** that cross-references multiple public record sources:

| Signal | Source | Severity |
|--------|--------|----------|
| Late property taxes | County tax collector | HIGH |
| Foreclosure filing (lis pendens) | County clerk | CRITICAL |
| Partition action | County clerk | HIGH |
| Unresolved code violations | City code enforcement | MEDIUM |
| Dissolved corporate entity | State registry (Sunbiz) | MEDIUM |
| Multiple recent UCC filings | Secretary of State | MEDIUM |
| Mortgage maturing within 12 months | County clerk | HIGH |
| Owner age 70+ | Enformion background check | MEDIUM |
| Out-of-state owner | Enformion / county appraiser | MEDIUM |
| Property tax appeal filed | County appraiser | LOW-MEDIUM |
| Owner bankruptcy | PACER | CRITICAL |
| Estate/probate filing | County clerk | HIGH |
| CMBS special servicing | Trustee reports | CRITICAL |

When multiple distress signals stack on a single property, the system escalates it automatically. The benchmark case was a 150,000 SF heavy industrial property in Clearwater, Florida — Spectre traced the ownership chain, found 39 associated addresses, uncovered a $22 million judgment against the owner, and flagged the entity as being in receivership. That is the kind of intelligence that typically requires a private investigator and several days. Axon found it in minutes.

### 2.6 The Ownership Chain

Tracing who actually owns a commercial property is harder than it sounds. Properties are almost never held in personal names — they sit inside LLCs, which are managed by other LLCs, which are owned by trusts or holding companies.

Axon's ownership tracing chain:

1. **County Records** — Pull the deed. Get the entity name on title.
2. **State Registry** — Search the entity in the state's corporate database (Sunbiz for Florida, SOS for California). Get the officers, members, and registered agent.
3. **Entity Chain Tracing** — If the officers are themselves LLCs, trace those entities. Repeat until you reach a human name.
4. **Background Check** — Run the human through Enformion. Get full name, age, date of birth, 37+ historical addresses with dates, phone numbers, emails, relatives, associates, and other business records.
5. **SEC Cross-Reference** — If the entity structure suggests institutional or REIT involvement, check SEC EDGAR filings.

This chain runs automatically for every deal that enters the deep dive stage. The result is a complete dossier on who owns the property, what else they own, and whether they might be motivated to sell.

## 3. The Numbers

### 3.1 Throughput

The system's first major batch run processed **13,650 properties in 36 hours**. This included ingesting property data, scoring each against investment criteria, and generating summary analyses for every property above threshold.

The database grew to **12,360+ deals** with full structured data. Of those, 249 generated report cards, 82 received full nine-agent deep dives, and 20 entered the active pipeline.

Equivalent human effort at 4 hours per deal for basic screening: **54,600 hours — 26 analyst-years.**

### 3.2 Cost

| Component | Cost |
|-----------|------|
| Cloud server (8GB RAM, 4 vCPU) | $48/month |
| Mac Mini M4 (browser automation node) | $599 one-time |
| AI extraction per document (BRAID 3-pass) | $0.004 |
| Deep agent analysis per deal (all 9 agents) | $3-8 |
| Enformion background checks | $0.35/search |
| Total AI cost for first 197 verified deals | $0.83 |

For context: a junior CRE analyst costs $65,000-85,000/year plus benefits and can process maybe 2-3 deals per day thoroughly. The system processes hundreds per day at roughly 1% of the cost.

### 3.3 Quality Assurance

The system includes a seven-layer quality harness:

1. **Unit Tests** — 155 automated tests on individual functions
2. **Integration Tests** — End-to-end flow across 7 property types
3. **Regression Tests** — Output compared against 10 saved baselines
4. **Simulation Battery** — 29/31 stress tests passing (concurrency, recovery, adversarial input)
5. **Grade Pipeline** — 5-layer automated quality scoring (completeness, sanity, cross-reference, regression, audit)
6. **Visual QA** — Computer vision opens output files, takes screenshots, verifies values against source documents
7. **Adversarial Tests** — 13/13 passing (malicious PDFs, injection attempts, edge cases)

Every agent is graded after every pipeline run. Any agent that scores below A- is automatically diagnosed, has its instructions adjusted, and is re-run — no human intervention required.

## 4. How It Actually Works

### 4.1 Infrastructure

Axon runs 24/7 on a DigitalOcean VPS — Ubuntu 24.04, 8GB RAM, 4 vCPUs. It is not a chatbot that waits for questions. It is an autonomous agent that checks in every 30 minutes, scans for new deals, runs pipeline tasks, monitors system health, and reports results to the team through Discord.

A Mac Mini M4 ($599) serves as a secondary node for browser automation. County record systems (property appraisers, clerks of court, tax collectors) block datacenter IP addresses — the Mac Mini provides a residential IP and a real browser environment for accessing these systems.

### 4.2 The AI Model Stack

| Task | Model | Cost |
|------|-------|------|
| Email classification, routing, quick categorization | Claude Haiku | ~$0.01/task |
| Document extraction, deal scoring, standard analysis | Claude Sonnet | ~$0.004/document |
| Complex underwriting, synthesis, ownership intelligence | Claude Opus | ~$2-5/analysis |

Prompt caching reduces costs ~90% for repeated patterns.

### 4.3 Document Extraction (BRAID)

For deals with offering memoranda — the dense PDF packages that brokers send — we built a three-pass extraction system:

**Pass 1 — Extract.** The model reads the full document and pulls 20 structured fields: property name, address, city, state, zip, property type, year built, total square footage, lot size, asking price, cap rate, NOI, price per square foot, occupancy, number of tenants, largest tenant, weighted average lease term, clear height, dock doors, and parking spaces. Each field gets a confidence score.

**Pass 2 — Verify.** A second pass with a different prompt checks whether every extracted number actually appears in the source document. Disagreements between Pass 1 and Pass 2 are flagged.

**Pass 3 — Derive.** Missing fields are calculated where mathematically possible (price and cap rate known → derive NOI). Fields that cannot be verified or derived are marked absent — never estimated.

### 4.4 The Agent Intelligence System

Each agent learns from its work:

- **Memory**: Before starting any analysis, agents recall prior learnings from a persistent database (252+ learnings stored, 22+ shared cross-agent insights)
- **Agent-to-Agent Messaging**: Agents request information from each other mid-pipeline via a message bus
- **Auto-Grading**: After every run, agents are graded A+ through F — below A- triggers automatic diagnosis and re-run
- **Dynamic Tool Creation**: When an agent needs a data source that doesn't exist yet, it can generate a new API integration script from a 73+ tool registry

## 5. Why This Matters Beyond Real Estate

The architecture we built is not specific to commercial real estate. The pattern — high-volume document intake, structured data extraction, multi-source enrichment, configurable scoring, specialist agents with domain tools — applies to any industry where professionals spend their time manually processing documents and making structured decisions.

### 5.1 The General Pattern

Any business that matches these criteria is a candidate:

1. **High-volume document intake** — more documents arrive than humans can thoroughly review
2. **Structured decision-making** — the decision can be broken into measurable criteria
3. **External data enrichment** — public or licensed data sources add context beyond the documents
4. **Specialist knowledge required** — the analysis needs domain expertise that is expensive to hire
5. **Historical outcomes** — past decisions can be compared against what actually happened

### 5.2 Life Settlements and Insurance Underwriting

Consider a firm that acquires life insurance policies on the secondary market. The underwriting process involves reviewing medical records — sometimes hundreds, sometimes 20,000 pages per individual — to forecast how long someone will live. The firm collects updated medical records every year and re-evaluates each policy.

This maps directly onto the Axon architecture:

| Axon (CRE) | Life Settlement Equivalent |
|------------|---------------------------|
| Offering memoranda (20-200 pages) | Medical records (100-20,000 pages) |
| 20 structured fields extracted | Diagnoses, medications, lab values, vital sign trends |
| 9 specialist agents | Virtual specialist panel (oncologist, cardiologist, neurologist, etc.) |
| County records, FEMA, Census, FRED | Actuarial tables, clinical databases, pharmaceutical databases |
| Deal scoring (0-100) | Life expectancy forecast |
| Distress signals (liens, foreclosure) | Condition progression (lab trends, medication changes) |
| Human override → system learns | Actual mortality outcomes → strongest possible training signal |

The critical advantage for life settlements: **you know the actual outcome.** In real estate, it takes years to know if a deal was good. In life settlements, the outcome is definitive — the prediction was right or it was wrong. A firm with 20 years of historical data — policies purchased, predictions made, actual outcomes observed — possesses one of the most valuable training datasets imaginable.

The privacy constraint (HIPAA) is solved the same way we solved county record access — with local hardware. A Mac Mini in a closet, running AI models locally, all patient data staying on-premises. No data leaves the building.

### 5.3 Other Applications

- **Credit underwriting** — loan applications instead of OMs, credit bureau data instead of county records
- **Legal due diligence** — contract review, regulatory filings, litigation risk
- **Insurance claims** — damage assessment, fraud detection, comparable claim analysis
- **Pharmaceutical research** — clinical trial data extraction, adverse event monitoring
- **Regulatory compliance** — document review, violation detection, remediation tracking

## 6. What We Learned

**Start with the data, not the model.** The most valuable thing we built was not the AI pipeline — it was the 130+ tool integrations that feed it real data. County tax records, corporate registries, background checks, flood maps, construction permits — that is what turns a document summarizer into an intelligence system.

**Agents need specialization.** A single general-purpose AI trying to do everything produces mediocre analysis. Nine specialists, each with deep domain knowledge and curated tools, produce work that looks like it came from a team of senior analysts.

**The quality harness is the product.** Anyone can get an AI to produce a good analysis once. The engineering challenge is making it reliable every time, across thousands of documents, without human oversight. Seven layers of testing is what makes the difference between a demo and a system you can trust.

**Two people can build what used to require a team.** Axon was built by two people in six months. It does the work of 26 analysts. The gap between what a small team can build with AI and what used to require an entire department is wider than most people realize — and it is getting wider every month.

---

## About the Team

**Pierce Millar** — COO. Business development, workflow analysis, client relationships. Founded the Dawn's Business Institute at Cathedral Catholic High School. San Diego, CA.

**Jackson Kiil** — CEO. System architecture, AI engineering, pipeline development. Stanford '25. Oakland, CA.

**Keegan Tingey** — CMO. Growth strategy, market positioning, operations.

**Axon** — The ninth agent. Runs 24/7. Never sleeps. Checks in every 30 minutes. Has processed 12,360+ deals and counting.

Contact: hello@prelium.ai | [prelium.ai](https://prelium.ai)

---

*This document describes a system in active development and production use. All statistics are from verified internal records as of April 2026. Technical details have been simplified for accessibility.*
