# ATM Network Risk Intelligence

**Operational risk framework for ATM cash management — built on the insight that volume is the wrong primary signal.**

---

## The Problem With Standard ATM Reporting

Most ATM dashboards answer one question: *where is withdrawal demand highest?*

That question produces useful reporting. It does not produce operational intelligence.

I know this because I spent years managing ATM networks. Every Sunday, I would log in and order emergency replenishments on terminals that standard reporting had no framework to prioritize correctly. Penn Station, New York — four Cardtronics terminals side by side, each loaded with $150,000 on Friday. All four would burn to zero by Sunday night. The reality was $600,000 in collective cash exposure burning down over 48 hours with Monday morning rush hour coming.

These were bank-branded terminals. That adds a layer of risk that pure cash management metrics never capture. A bank-branded ATM running out of cash is not just an operational failure — it is a brand failure. Every commuter who walks up to that machine on a Monday morning and sees an out-of-service screen associates that failure with the bank whose name is on it. Reputational consequence does not show up in a replenishment cost report. It does not appear in a transaction volume dashboard. But it is real, and it raises the stakes of every cash-out event significantly.

No standard dashboard flagged any of this. The operator had to know it.

**This project builds the framework that makes that knowledge explicit and scalable.**

---

## What Version 2 Established

[ATM Network Analysis — Version 2](https://github.com/SEANSKIDATA/ATM-Network-Analysis-Version-2) answered the volume question across a simulated national network:

- Which regions have the highest cash demand?
- Which terminals are exceeding their daily targets?
- Where should replenishment cycles be prioritized?

That work exposed the limitation. Volume tells you what already happened. It does not tell you what is about to go wrong — or how expensive it will be when it does.

---

## What Risk Intelligence Adds

This project introduces four variables that standard ATM reporting ignores entirely:

| Variable | Why It Matters |
|---|---|
| `terminal_type` — Local / Remote / Over The Road | Distance determines emergency response time and cost |
| `cash_tolerance` — Zero / Low / Medium / High | A casino cannot absorb even one hour of downtime |
| `emergency_multiplier` — 2.5x to 4.0x | An Over The Road emergency fill costs 4x a scheduled run |
| `days_until_empty` | Forward-looking runway, not backward-looking volume |

The result is a **Network Criticality Priority Register** — a ranked list of every ATM in the network ordered by operational consequence, not transaction count.

---

## The Core Insight

> A casino ATM 142 miles from the nearest branch with 0.4 days of cash remaining is a five-alarm emergency — regardless of how many transactions it processes.
>
> A high-volume urban transit ATM with 6 days of cash runway can wait.
>
> Standard reporting ranks them in the wrong order. This framework corrects that.

---

## Cluster Risk — A Concept Standard Reporting Misses Entirely

One of the most significant findings in this dataset involves terminal clustering.

**Penn Station, New York City — 4 co-located terminals:**

| Metric | Individual Terminal | Cluster (4 terminals) |
|---|---|---|
| Cash capacity | $150,000 | $600,000 |
| Friday load | $150,000 | $600,000 |
| Cash by Sunday night | ~$0 | ~$0 |
| Emergency fills Q1 | ~13 | ~52 |
| Criticality tier (individual) | MEDIUM | **CHRONIC EMERGENCY PATTERN** |

Standard reporting sees four separate low-risk local machines. Risk Intelligence sees a single-point-of-failure cluster deploying $600,000 every Friday and burning to zero every weekend.

The `cluster_location_id` field in `atm_locations` groups co-located terminals so Query 6 can surface this exposure explicitly.

---

## Dataset

**Scope:** 50+ ATMs across 5 regions — Northeast, Southeast, Midwest, Southwest, West  
**Time period:** Q1 2026 — January 1 through March 31 (90-day time series)  
**Granularity:** Daily transaction records + weekly summaries  
**Platform:** MySQL  

The dataset is purpose-built synthetic data designed to reflect realistic ATM network operating conditions including:

- Regional demand variance
- Location-type performance differences (Casino vs Airport vs Transit vs Retail)
- Distance-based replenishment economics
- Weekend vs weekday demand patterns
- Cash target thresholds and tolerance levels by location type

*Synthetic data is disclosed here and throughout the project. The operational logic, cost structures, and risk variables are derived from real-world ATM network management experience.*

---

## Data Model

### `atm_locations` — Static Reference
Master ATM reference table. Every terminal's physical, operational, and risk profile.

Key fields: `terminal_type`, `cash_tolerance`, `distance_from_branch_miles`, `emergency_multiplier`, `revenue_impact_per_hour_usd`, `cluster_location_id`

### `atm_transactions` — 90-Day Time Series
Daily transaction records for all 50+ ATMs across Q1 2026.

Key fields: `total_cash_dispensed_usd`, `opening_cash_usd`, `closing_cash_usd`, `replenishment_type`, `days_since_last_fill`, `is_weekend`

### `replenishment_schedule` — Cost Modeling
Every replenishment event — scheduled and emergency — with full cost breakdown.

Key fields: `replenishment_type`, `scheduled_cost_usd`, `actual_cost_usd`, `cost_variance_usd`, `emergency_multiplier`, `response_time_hours`, `revenue_lost_usd`

### `atm_criticality` — Derived Risk Scores
End-of-quarter snapshot. Composite risk scoring, criticality tier assignment, and recommended actions.

Key fields: `risk_score`, `criticality_tier`, `days_until_empty`, `avg_daily_burn_usd`, `q1_emergency_count`, `q1_total_emergency_cost_usd`, `q1_total_revenue_lost_usd`, `recommended_action`

---

## The 7 Core Queries

### Query 1 — Network Criticality Priority Register
The primary output of the framework. Every ATM ranked by risk score. Not by volume.

### Query 2 — Over The Road Terminal Risk Summary
Isolates the highest-cost risk tier. Proves that distance × zero cash tolerance × emergency multiplier creates outsized operational exposure regardless of transaction count.

### Query 3 — Emergency vs Scheduled Cost by Region
Quantifies the true cost of reactive cash management. Shows how emergency premium rates vary by region and identifies where proactive scheduling generates the most savings.

### Query 4 — Cash Burn Rate vs Replenishment Frequency Gap
Forward-looking risk identification. Flags machines where burn rate is outpacing fill cycles before the outage happens.

### Query 5 — 90-Day Transaction Trend with Day-of-Week Patterns
Surfaces weekend vs weekday demand splits by location type. Proves why casino and transit ATMs spike on weekends and why that spike creates predictable risk.

### Query 6 — Cluster Risk Analysis
Groups co-located terminals by `cluster_location_id`. Calculates collective cash exposure, cluster burn rate, and cluster-level emergency patterns. Surfaces the $600K Penn Station problem that individual-terminal reporting cannot see.

---

## Key Findings

**1. Over The Road terminals account for disproportionate emergency cost**
Five Over The Road terminals in the dataset generated more emergency replenishment cost in Q1 2026 than all 30 Local terminals combined — despite processing less total transaction volume.

**2. Distance multiplies every other risk factor**
A casino ATM 156 miles from its nearest branch (Laughlin, NV) averaged 16+ hour emergency response times in Q1. The same cash tolerance event at a local Las Vegas casino resolved in under 2 hours. Same risk profile. Completely different operational consequence.

**3. Cluster risk is invisible to standard reporting**
Penn Station's four-terminal cluster generated 52 emergency fills in Q1 — every one of them predictable, every one of them avoidable with a proactive Sunday replenishment protocol. The operator who ordered Sunday fills was running a manual version of this framework. This project makes it automatic.

**4. High volume does not mean high risk**
The highest-volume terminals in the dataset — Bellagio, MGM Grand — are classified MEDIUM, not CRITICAL. Local proximity keeps their emergency costs manageable. The CRITICAL tier is dominated by remote and Over The Road terminals with moderate transaction volume and zero cash tolerance.

---

## Tools Used

- MySQL — data modeling, querying, all analytical logic
- Google Sheets — dashboard visualization
- Synthetic dataset — purpose-built to reflect realistic operating conditions

---

## Progression From Version 2

| | Version 2 | Risk Intelligence |
|---|---|---|
| Core question | Where is demand highest? | Which machines can't afford to fail? |
| Risk signal | Volume + target variance | Distance + tolerance + days until empty |
| Decision output | Replenishment optimization by region | Criticality priority register |
| Analytical approach | Descriptive | Predictive / operational |
| Data model | 3 flat CSVs | 4 relational tables with derived variables |
| New concept | — | Cluster Risk |

---

## About

Built by **Sean** — Logistics Analyst and data analytics portfolio developer.  
GitHub: [SEANSKIDATA](https://github.com/SEANSKIDATA)

*This project reflects operational thinking developed through real-world ATM network management experience, translated into a reusable analytical framework.*

### Query 7 — Bank-Branded Terminal Reputational Risk Register
Introduces a risk dimension that no standard ATM report captures: brand exposure layered on top of operational exposure. An independent ATM running out of cash is an operational failure. A bank-branded ATM running out of cash is a brand failure. This query ranks bank-branded terminals by combined operational and reputational risk, applies a brand impact multiplier to outage cost estimates, and flags terminals where cash-out events carry institutional consequence beyond direct revenue loss.

---

## Key Finding — Brand Type Creates a Hidden Risk Multiplier

Bank-branded terminals at high-footfall transit locations carry reputational exposure that does not appear in any standard cash management report. Penn Station's four bank-branded terminals averaged 13 emergency fills each in Q1 — every cash-out event a potential brand failure in front of thousands of commuters. Query 7 makes this exposure visible and quantifiable for the first time.

---

## Future Enhancements

- Predictive cash demand modeling using day-of-week and seasonal patterns
- Automated replenishment scheduling recommendations by terminal tier
- Integration of weather and event data as demand spike predictors
- Expansion of `brand_type` to include specific institution names for portfolio-level brand risk reporting
