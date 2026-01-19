# Carbon Mirror Standard v1.0
## Core Specification

**Status:** Canonical  
**Maintainer:** Carbon Mirror Foundation  
**License:** Creative Commons BY 4.0  
**Normative Language:** The terms MUST, SHALL, SHOULD, MAY are to be interpreted as in RFC 2119.

---

# 1. Purpose and Scope

## 1.1 Purpose

The Carbon Mirror Standard defines a normative, auditable, vendor-neutral specification for:

- Measuring operational inefficiency in digital systems  
- Quantifying avoidable energy, cost, and carbon waste  
- Enforcing continuous efficiency improvement  

The Standard is designed to:

- Make inefficiency visible  
- Make optimization measurable  
- Make greenwashing difficult or impossible  

---

## 1.2 Scope

This Standard applies to:

- Cloud, on-prem, hybrid, and edge systems  
- Compute, storage, data pipelines, and AI workloads  
- Organizations of any size  

This Standard does not mandate technologies. It defines outcomes, measurements, and verification requirements.

---

# 2. Normative References

The following documents are normative parts of this Standard:

- Carbon Mirror Telemetry API Specification  
- Carbon Mirror Auto-Certification Logic Specification  
- Carbon Mirror Level-1 Field Guide  

In case of conflict, this document takes precedence.

---

# 3. Definitions and Concepts

## 3.1 The Four Mirrors

All scoring and certification is based on four analytical perspectives:

1. Efficiency Mirror — Compute waste  
2. Timing Mirror — Carbon/time alignment  
3. Data Diet Mirror — Data mass and movement waste  
4. Circularity Mirror — Reuse and recovery  

---

## 3.2 Core Terms

**Workload:** A discrete unit of compute or processing.  
**Telemetry:** Machine-generated measurements of resource usage.  
**Mirror Score:** A normalized score in the range 0–100.  
**Composite Efficiency Score (CES):** Weighted aggregate score.  
**Evidence Package:** The mandatory audit bundle.

---

# 4. Metrics and Measurement (NORMATIVE)

## 4.1 Measurement Principles

Measurements MUST be reproducible, continuous, attributable, comparable, and auditable.

---

## 4.2 Required Telemetry Domains

- Compute utilization  
- Energy and power  
- Time and scheduling  
- Data movement and storage  
- Waste and reuse signals  

---

## 4.3 Mirror Scores

### 4.3.1 Efficiency Mirror Score (EMS)

```text
EMS = 100 × (1 - (Wasted_Compute / Total_Provisioned_Compute))
4.3.2 Timing Mirror Score (TMS)
TMS = 100 × (1 - (Carbon_Intensity_Weighted_Runtime / Worst_Case_Runtime))
4.3.3 Data Diet Mirror Score (DMS)
DMS = 100 × (1 - (Unnecessary_Data_Moved / Total_Data_Moved))
4.3.4 Circularity Mirror Score (CMS)
CMS = 100 × (Reclaimed_or_Avoided_Waste / Total_Potential_Waste)
4.4 Composite Efficiency Score (CES)
CES = 0.35·EMS + 0.25·TMS + 0.20·DMS + 0.20·CMS
4.5 Measurement Windows
Scores MUST be computed as:

Real-time rolling

Daily

Monthly

Annual

Certification uses rolling 90-day performance.

4.6 Anti-Gaming Rules
Implementations MUST NOT:

Exclude workloads

Exclude time windows

Exclude failed jobs

Exclude background work

Manually manipulate scores

5. Compliance and Audit (NORMATIVE)
5.1 Certification Authority Model
Bronze MAY be self-attested

Silver and Obsidian MUST be audited or automated

5.2 Evidence Package (MANDATORY)
Must include:

Inventory

Telemetry

Mirror scores

CES outputs

Configurations

Audit logs

Retention: ≥ 24 months.

5.3 Audit Types
Desk audit

Systems audit

Surprise audit (Obsidian)

5.4 Non-Compliance
Any telemetry gaps, manipulation, or exclusion = non-compliance.

5.5 Decertification
Fraud or repeated failure = revocation.

5.6 Public Claims
All claims MUST include scope, date, window, and tier.

6. Certification Tiers
6.1 Bronze
≥90% inventory

≥90% telemetry

Audit complete

6.2 Silver
All Bronze

Dashboard

≥5% YoY CES improvement

Verification required

6.3 Obsidian
Continuous telemetry

Automated scoring

Public reporting

Surprise audit eligible

7. Governance and Versioning
Semantic versions

Normative changes require major version

8. Intellectual Property
Standard text: CC BY 4.0

9. Conformance
Claims MUST specify:

“Carbon Mirror Standard v1.0 — SaaS Profile — Silver — 2026-Q2”

10. Industry Profiles (NORMATIVE)
10.1 Profile Applicability
If a system matches a profile, it SHALL apply. Multiple profiles may apply. The strictest rules prevail.

10.2 AI / ML Profile
Applies to systems where ≥25% of compute is ML-related.

Additional telemetry: GPU hours, VRAM utilization, tokens processed, failed runs, reuse ratio.

Weights:

EMS 0.40 | TMS 0.20 | DMS 0.25 | CMS 0.15
Level 2 Gates:

30% idle GPU = FAIL

25% discarded runs = FAIL

10.3 SaaS / Web Platform Profile
Applies to SaaS, APIs, web platforms.

Additional telemetry: overprovisioning, requests per action, cache hit, fanout, background jobs.

Weights:

EMS 0.35 | TMS 0.15 | DMS 0.30 | CMS 0.20
Level 2 Gates:

Overprovisioning >2.5× = FAIL

Cache hit <60% = FAIL

10.4 Data Platform Profile
Applies to warehouses, ETL, BI, analytics.

Additional telemetry: bytes scanned, rebuilds, unused data, duplicate data, dashboard waste.

Weights:

EMS 0.30 | TMS 0.20 | DMS 0.35 | CMS 0.15
Level 2 Gates:

40% data never read = FAIL

30% pipelines are rebuilds = FAIL

10.5 HPC / Data Center Profile
Applies to on-prem, private cloud, HPC, colocation.

Additional telemetry: PUE, rack power, idle nodes, scheduling efficiency, cooling energy.

Weights:

EMS 0.40 | TMS 0.20 | DMS 0.15 | CMS 0.25
Level 2 Gates:

Utilization <45% = FAIL

25% idle powered-on nodes = FAIL

11. Level 2 — Precision Layer (Silver & Obsidian Only)
11.1 Gating Rule
For Silver: no Mirror Score < 40

For Obsidian: no Mirror Score < 60

11.2 Anti-Compensation Rule
CES does not override gating failures.

11.3 Thresholds
Tier	CES	Min Mirror
Silver	60	40
Obsidian	80	60
12. Canonicalization
This document constitutes Carbon Mirror Standard v1.0.
Any change to formulas, weights, or gates requires v2.0.

13. Conformance Statement
“This system conforms to Carbon Mirror Standard v1.0 under the SaaS Profile and Level 2 rules, achieving Silver certification for the period 2026-Q2.”
