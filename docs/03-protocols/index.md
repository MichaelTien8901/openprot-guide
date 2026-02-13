---
layout: default
title: "3. Protocols"
nav_order: 4
has_children: true
---

# Core Protocols

The three protocols form a stack where **MCTP** is the foundation:

```mermaid
graph TD
    PLDM["PLDM<br/>(FW Update · Monitoring)"] --> MCTP
    SPDM["SPDM<br/>(Attestation · Security)"] --> MCTP
    MCTP["MCTP<br/>(Message Transport Layer)"] --> SMBus["SMBus/I2C"]
    MCTP --> PCIe["PCIe-VDM"]
    MCTP --> USB["USB"]

    style PLDM fill:#4a70a0,color:#fff
    style SPDM fill:#509060,color:#fff
    style MCTP fill:#a09825,color:#fff
    style SMBus fill:#a05050,color:#fff
    style PCIe fill:#a05050,color:#fff
    style USB fill:#a05050,color:#fff
```

## Protocol Deep Dives

| Protocol | Role | Details |
|----------|------|---------|
| [MCTP](mctp.md) | Transport Layer | Message delivery between components |
| [SPDM](spdm.md) | Security & Attestation | Identity verification and trust |
| [PLDM](pldm.md) | Firmware & Monitoring | Management operations |

## Key Takeaway

- **MCTP** = How messages travel (transport)
- **SPDM** = Who you're talking to and can you trust them (security)
- **PLDM** = What you're actually doing (management operations)

---

[Prev: Architecture](../02-architecture.md) | [Next: MCTP](mctp.md)
