# Services Layer Deep Dive

## Attestation Service

Provides cryptographic proof of platform identity and firmware integrity.

**Capabilities:**
- SPDM Responder — proves identity to external verifiers
- SPDM Requester — verifies downstream devices
- Local Verifier — on-platform trust decisions (for air-gapped environments)
- Evidence format: RATS EAT (Entity Attestation Token) with OCP profile

### DICE Integration

The PRoT uses a DICE (Device Identifier Composition Engine) chain to build layered measurements from hardware through each firmware stage:

```mermaid
graph LR
    HW["Hardware"] -->|CDI| BR["Boot ROM"]
    BR -->|CDI| S1["Stage 1 FW"]
    S1 -->|CDI| S2["Stage 2 FW"]
    S2 -->|CDI| RT["Runtime"]
    RT -->|CDI| FI["Final Identity"]

    style HW fill:#a05050,color:#fff
    style BR fill:#a09825,color:#fff
    style S1 fill:#509060,color:#fff
    style S2 fill:#4a70a0,color:#fff
    style RT fill:#804080,color:#fff
    style FI fill:#705090,color:#fff
```

Each stage measures the next and derives a new Compound Device Identifier (CDI), creating an unforgeable chain of trust.

## Firmware Update & Recovery

Follows NIST SP 800-193 (Platform Firmware Resiliency):

- **Dual-bank storage** — Active bank + recovery bank
- **Authenticated packages** — Cryptographic signature verification
- **Rollback protection** — Anti-rollback counters prevent downgrade attacks
- **Automatic recovery** — Falls back to known-good bank on boot failure

## Telemetry Service

Platform monitoring via PLDM Type 2:
- Temperature, voltage, and power sensors
- Platform Data Records (PDR) repository
- Standardized sensor descriptions for management tools

---

[Prev: Tutorial: Secure Firmware Update](08-tutorial-secure-firmware-update.md) | [Next: Standards & Compliance](10-standards-and-compliance.md)
