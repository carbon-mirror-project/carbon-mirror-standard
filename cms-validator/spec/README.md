# CMS Validator

The **CMS Validator** is the canonical conformance engine for the **Carbon Mirror Standard v1.0**.

It is responsible for deterministically evaluating telemetry data, enforcing anti-gaming rules, computing efficiency scores, and assigning certification tiers.  
**The Validator is the authority.** Human interpretation and dashboards MUST NOT override its results.

---

## What This Repository Contains

This repository provides:

- The **Validator Specification** (normative)
- Reference schemas and test vectors
- A reference implementation (planned / in progress)
- Tooling to validate Carbon Mirror conformance in CI/CD and audits

---

## Canonical Specification (START HERE)

📘 **Validator Specification v1.0**  
➡️ [`spec/Validator_Specification_v1.0.md`](spec/Validator_Specification_v1.0.md)

This document defines:

- Required inputs (telemetry bundles and profiles)
- Deterministic validation phases
- Gating rules and scoring logic
- Certification tier assignment
- Cryptographic attestation requirements
- Exit codes and reproducibility guarantees

If there is ever a discrepancy between code, documentation, or dashboards,  
**the specification takes precedence**.

---

## How the Validator Fits Into Carbon Mirror

The CMS Validator sits between **raw telemetry** and **public certification claims**.

```
Telemetry Sources
   ↓
Telemetry Bundle (JSON)
   ↓
CMS Validator  ←── this repo
   ↓
Validation Report + Cryptographic Attestation
   ↓
Certification Tier / Public Claims
```

Key principles:

- Telemetry is **pulled**, not self-reported
- All scoring is **deterministic**
- Anti-gaming rules are enforced in code
- Outputs are **auditable and reproducible**

---

## Status

- ✅ Validator Specification v1.0 — **Canonical**
- 🚧 Reference Implementation — **In Progress**
- 🚧 Golden Test Vectors — **Planned**
- 🚧 CLI Tooling — **Planned**

---

## Planned Repository Structure

```
cms-validator/
├── README.md
├── spec/
│   └── Validator_Specification_v1.0.md
├── schemas/
│   └── telemetry/
├── test_vectors/
│   └── v1.0/
├── src/
│   └── cms_validator/
└── cli/
    └── cms-validator
```

---

## Intended Usage

The Validator is designed to be used in:

- Certification audits
- Continuous compliance pipelines
- Internal efficiency governance
- Independent third-party verification

Typical invocation (future CLI):

```bash
cms-validator validate   --telemetry ./telemetry   --profile profile.json   --out ./results
```

---

## Governance & Versioning

- Validator versions are tied to **Carbon Mirror Standard major versions**
- Validator v1.x SHALL NOT change formulas or thresholds
- Any change that alters scoring outcomes requires **CMS v2.0**
- Backward compatibility and deprecation rules are defined in the spec

---

## Contributing

This repository follows the governance rules of the **Carbon Mirror Standard**.

Contributions are welcome for:

- Reference implementations
- Test vectors
- Schema validation
- Documentation improvements

All contributions MUST preserve:

- Determinism
- Auditability
- Vendor neutrality
- Anti-gaming guarantees

---

## License

- **Specification:** Apache License 2.0  
- **Reference Code:** Apache License 2.0  

---

## Final Note

The CMS Validator exists to make one principle enforceable:

> **Claims are optional. Evidence is mandatory.**

If the Validator says a system is compliant, it is.  
If it does not, no amount of narrative changes that.
