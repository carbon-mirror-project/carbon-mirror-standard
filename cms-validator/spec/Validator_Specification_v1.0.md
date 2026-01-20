# Carbon Mirror Standard
## Validator Specification v1.0

**Status:** Canonical  
**Maintainer:** Carbon Mirror Foundation  
**Applies to:** Carbon Mirror Standard v1.0  
**License:** Apache License 2.0  

**Normative Language:**  
The terms **MUST**, **SHALL**, **SHOULD**, and **MAY** are to be interpreted as described in RFC 2119.

---

## Table of Contents

1. [Purpose and Authority](#1-purpose-and-authority)
   - [Purpose](#11-purpose)
   - [Authority](#12-authority)
2. [Scope and Non-Scope](#2-scope-and-non-scope)
   - [In Scope](#21-in-scope)
   - [Out of Scope](#22-out-of-scope)
3. [Inputs](#3-inputs)
   - [Telemetry Bundle (REQUIRED)](#31-telemetry-bundle-required)
   - [Validation Profile (REQUIRED)](#32-validation-profile-required)
4. [Deterministic Validation Pipeline](#4-deterministic-validation-pipeline)
   - [Phase 1 — Structural Validation](#phase-1--structural-validation-mandatory)
   - [Phase 2 — Plausibility and Anti-Gaming Checks](#phase-2--plausibility-and-anti-gaming-checks)
   - [Phase 3 — Metric Calculation](#phase-3--metric-calculation)
   - [Phase 4 — Gating Rules Enforcement](#phase-4--gating-rules-enforcement)
   - [Phase 5 — Composite Efficiency Score (CES)](#phase-5--composite-efficiency-score-ces)
   - [Phase 6 — Tier Assignment](#phase-6--tier-assignment)
5. [Outputs](#5-outputs)
   - [Validation Report (REQUIRED)](#51-validation-report-required)
   - [Cryptographic Attestation (MANDATORY)](#52-cryptographic-attestation-mandatory)
6. [Exit Codes](#6-exit-codes-cli-implementations)
7. [Determinism and Reproducibility](#7-determinism-and-reproducibility)
8. [Versioning and Compatibility](#8-versioning-and-compatibility)
9. [Security Considerations](#9-security-considerations)
10. [Conformance Statement](#10-conformance-statement)
11. [Canonicalization](#11-canonicalization)

---



## 1. Purpose and Authority

### 1.1 Purpose

This document defines the **Carbon Mirror Standard Validator (CMS Validator)** — the canonical, deterministic conformance engine used to evaluate telemetry data against Carbon Mirror Standard v1.0.

The CMS Validator is the **authoritative mechanism** for:

- Determining conformance
- Computing Mirror Scores
- Applying gating rules
- Calculating the Composite Efficiency Score (CES)
- Assigning certification tiers
- Producing auditable, cryptographically verifiable results

### 1.2 Authority

For Carbon Mirror Standard v1.0:

> **Validator output SHALL be considered the final and authoritative determination of certification status.**

Human interpretation, dashboards, or secondary tools **MUST NOT** override Validator results.

---

## 2. Scope and Non-Scope

### 2.1 In Scope

The Validator SHALL:

- Validate telemetry completeness and structure
- Enforce anti-gaming rules
- Perform deterministic calculations defined in:
  - Carbon Mirror Standard v1.0
  - Telemetry API Specification v1.0
- Assign certification tiers
- Emit signed validation artifacts

### 2.2 Out of Scope

The Validator SHALL NOT:

- Collect telemetry
- Estimate missing data
- Modify formulas or weights
- Perform scheduling or remediation actions
- Persist long-term state (grace periods handled externally)

---

## 3. Inputs

### 3.1 Telemetry Bundle (REQUIRED)

The Validator SHALL accept a **Telemetry Bundle**, consisting of machine-readable files conforming to the Telemetry API Specification v1.0.

### Required files

```
ghost_compute.json
model_efficiency.json
dark_data.json
coverage_verification.json
```

### Optional files (conditionally required)

```
heat_recovery.json  (REQUIRED if thermal_load_kw ≥ 50)
```

All telemetry files MUST:

- Be timestamped
- Represent the declared measurement window
- Include source identifiers
- Be complete (no cherry-picking)

---

### 3.2 Validation Profile (REQUIRED)

The Validator SHALL require an explicit Validation Profile declaring the evaluation context.

#### Example

```json
{
  "company_id": "acme-corp",
  "profile": "saas",
  "cms_version": "1.0",
  "measurement_window": "rolling-90d",
  "thermal_load_kw": 120,
  "declared_exemptions": []
}
```

The Validator MUST reject submissions lacking a Validation Profile.

---

## 4. Deterministic Validation Pipeline

The Validator SHALL execute the following phases **in order**.  
Failure in earlier phases SHALL prevent execution of later phases.

### Phase 1 — Structural Validation (MANDATORY)

The Validator SHALL verify:

- JSON schema conformance
- Presence of all required fields
- Timestamp freshness
- Telemetry coverage thresholds per Core Spec

**Failure Outcome**

- ❌ **Invalid Submission**
- No scores or tiers computed

---

### Phase 2 — Plausibility and Anti-Gaming Checks

The Validator SHALL perform plausibility analysis, including but not limited to:

- Utilization bounds checking
- Energy value sanity checks
- Sudden pattern shifts
- AI task classification consistency
- Dark data access anomalies

Detected anomalies SHALL be recorded as **Flags**.

Flags:

- DO NOT alter scores automatically
- MUST be included in Validator output
- MAY trigger audits externally

---

### Phase 3 — Metric Calculation

The Validator SHALL compute all applicable Mirror Scores using **only** formulas defined in Carbon Mirror Standard v1.0.

Computed metrics MAY include:

- Efficiency Mirror Score (EMS)
- Timing Mirror Score (TMS)
- Data Diet Mirror Score (DMS)
- Circularity Mirror Score (CMS)
- Heat Recovery Score (if applicable)

No weighting SHALL be applied at this phase.

---

### Phase 4 — Gating Rules Enforcement

The Validator SHALL enforce:

- Industry profile–specific Level-2 gates
- Minimum Mirror Score thresholds
- Anti-compensation rule

If any gating rule fails:

- The submission SHALL be marked as **GATING FAILURE**
- CES MAY be computed for informational purposes
- Tier assignment SHALL reflect the gating failure

---

### Phase 5 — Composite Efficiency Score (CES)

The Validator SHALL:

- Apply the correct profile-specific weights
- Compute CES deterministically
- Apply consistent rounding (two decimal places)

---

### Phase 6 — Tier Assignment

The Validator SHALL assign a certification tier based on:

- CES thresholds
- Gating outcomes
- Profile applicability

Grace periods, probation, and historical transitions are **explicitly excluded** from Validator logic and handled externally.

---

## 5. Outputs

### 5.1 Validation Report (REQUIRED)

The Validator SHALL emit a machine-readable Validation Report.

```json
{
  "cms_version": "1.0",
  "validator_version": "1.0.0",
  "company_id": "acme-corp",
  "profile": "saas",
  "measurement_window": "2026-01-01 to 2026-03-31",
  "mirror_scores": {
    "efficiency": 94.2,
    "timing": 88.7,
    "data_diet": 89.1,
    "circularity": 91.4
  },
  "ces": 90.6,
  "tier": "Obsidian Prime",
  "gating_failures": [],
  "flags": [],
  "exemptions_applied": [],
  "generated_at": "2026-04-01T00:12:11Z"
}
```

---

### 5.2 Cryptographic Attestation (MANDATORY)

Each Validation Report SHALL be accompanied by a cryptographic attestation.

```json
{
  "hash": "e3b0c44298fc1c149afbf4c8996fb924...",
  "signature": "MEUCIQDf9s1...",
  "algorithm": "ECDSA-P256"
}
```

This attestation SHALL serve as the **Evidence Package anchor**.

---

## 6. Exit Codes (CLI Implementations)

| Code | Meaning |
|----|--------|
| 0 | Valid submission, tier assigned |
| 10 | Valid submission with flags |
| 20 | Invalid submission |
| 30 | Gating failure |
| 40 | Internal Validator error |

---

## 7. Determinism and Reproducibility

The Validator MUST be:

- Deterministic
- Stateless
- Reproducible given identical inputs

Any change that alters scoring outcomes SHALL require a new **major CMS version**.

---

## 8. Versioning and Compatibility

- Validator versions SHALL align with CMS major versions
- Validator v1.x SHALL NOT alter formulas or thresholds
- Backward-incompatible changes require CMS v2.0
- Deprecated telemetry fields SHALL be supported for ≥90 days

---

## 9. Security Considerations

Validator implementations MUST:

- Reject unsigned or tampered telemetry bundles
- Protect signing keys using industry best practices
- Log all validation executions immutably

---

## 10. Conformance Statement

A system claiming conformance SHALL state:

> “Validated by CMS Validator v1.0 against Carbon Mirror Standard v1.0.”

---

## 11. Canonicalization

This document constitutes the **Carbon Mirror Standard Validator Specification v1.0**.

Any change affecting:

- Calculation logic
- Gating behavior
- Tier thresholds
- Attestation format

SHALL require **Validator Specification v2.0**.
