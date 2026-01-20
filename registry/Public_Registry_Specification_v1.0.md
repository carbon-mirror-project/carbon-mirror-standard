# Carbon Mirror Standard
## Public Registry Specification v1.0

**Status:** Canonical  
**Maintainer:** Carbon Mirror Foundation  
**Applies to:** Carbon Mirror Standard v1.0  
**License:** Apache License 2.0  

**Normative Language:**  
The terms **MUST**, **SHALL**, **SHOULD**, and **MAY** are to be interpreted as described in RFC 2119.

---

## 1. Purpose

The Carbon Mirror Public Registry defines the **canonical, read-only representation** of certification outcomes produced by the CMS Validator.

The registry exists to:

- Make certification claims verifiable
- Enable public transparency
- Prevent greenwashing
- Allow third-party dashboards without authority drift

> **The registry displays results.  
> It does not calculate, infer, or override them.**

---

## 2. Authority Model

1. The **CMS Validator** is the sole authority for certification outcomes.
2. The Public Registry SHALL only publish:
   - Validator outputs
   - Cryptographic attestations
3. The registry MUST NOT recompute scores or tiers.
4. Any conflict between registry display and validator output SHALL be resolved in favor of the validator.

---

## 3. Scope

### 3.1 In Scope

The Public Registry SHALL publish:

- Organization identity (limited)
- Certification tier
- Composite Efficiency Score (CES)
- Mirror Scores
- Measurement window
- Validation timestamp
- Attestation hash
- Certification history (optional)

### 3.2 Out of Scope

The Public Registry SHALL NOT publish:

- Raw telemetry
- Proprietary configurations
- Customer or user data
- Internal infrastructure topology
- Trade secrets
- Unvalidated or draft results

---

## 4. Registry Data Model (NORMATIVE)

### 4.1 Registry Entry (Canonical JSON)

Each certified organization SHALL have exactly one **current registry entry**.

```json
{
  "registry_version": "1.0",
  "cms_version": "1.0",
  "validator_version": "1.0.0",
  "organization": {
    "id": "examplecorp",
    "display_name": "Example Corp",
    "website": "https://example.com",
    "industry_profile": "saas"
  },
  "certification": {
    "tier": "Obsidian Prime",
    "composite_efficiency_score": 92.3,
    "mirror_scores": {
      "efficiency": 94.2,
      "timing": 88.7,
      "data_diet": 89.1,
      "circularity": 91.4
    },
    "measurement_window": {
      "start": "2026-01-01",
      "end": "2026-03-31",
      "type": "rolling-90d"
    },
    "validated_at": "2026-04-01T00:12:11Z"
  },
  "attestation": {
    "hash": "e3b0c44298fc1c149afbf4c8996fb924...",
    "algorithm": "SHA-256",
    "signature_algorithm": "ECDSA-P256"
  },
  "status": {
    "active": true,
    "revoked": false,
    "revocation_reason": null
  }
}
```

---

## 5. Certification History (OPTIONAL)

The registry MAY expose historical certification events.

```json
"history": [
  {
    "tier": "Obsidian Core",
    "score": 87.1,
    "validated_at": "2025-11-03T00:00:00Z"
  },
  {
    "tier": "Obsidian Prime",
    "score": 90.4,
    "validated_at": "2025-12-18T00:00:00Z"
  }
]
```

Historical entries MUST be immutable.

---

## 6. Public Display Requirements

Any public dashboard or viewer:

- MUST display tier and CES exactly as provided
- MUST display the validation timestamp
- MUST link to the attestation hash
- MUST indicate the CMS and Validator versions used

Dashboards MAY:

- Visualize trends
- Compare organizations
- Filter by profile or tier

Dashboards MUST NOT:

- Recalculate scores
- Modify weights
- Apply alternative thresholds
- Hide gating failures

---

## 7. Revocation and Status Changes

If a certification is revoked:

- The registry SHALL update `status.active` to `false`
- The previous certification SHALL remain visible
- The revocation reason SHALL be published

Silent removal of registry entries is prohibited.

---

## 8. Update Rules

- Registry entries SHALL only be updated by new validator attestations
- Partial updates are forbidden
- Registry consumers SHOULD treat entries as append-only records

---

## 9. Third-Party Dashboards

Third parties MAY:

- Mirror registry data
- Build alternative visualizations
- Host independent dashboards

Third parties MUST:

- Preserve registry semantics
- Attribute Carbon Mirror Standard
- Clearly state that dashboards are **non-authoritative**

---

## 10. Privacy and Safety

The registry is designed to publish **outcomes, not internals**.

If publication of a specific field creates legal or safety risk:

- That field MUST be omitted at the registry level
- The omission MUST be documented
- Certification validity MUST NOT be affected

---

## 11. Canonicalization

This document constitutes the **Carbon Mirror Public Registry Specification v1.0**.

Any change affecting:

- Published fields
- Authority boundaries
- Attestation handling
- Revocation visibility

SHALL require **Public Registry Specification v2.0**.

---

## Closing Note (Non-Normative)

The Public Registry exists to make efficiency claims durable, comparable, and falsifiable.

> **If it is not in the registry, it is not certified.**
