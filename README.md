# OpenPRoT: The Complete Guide

**Open Platform Root of Trust — An open-source firmware stack for Platform Root of Trust devices**

[![CHIPS Alliance](https://img.shields.io/badge/CHIPS-Alliance-blue)](https://www.chipsalliance.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green)](https://github.com/OpenPRoT/openprot/blob/main/LICENSE)
[![Rust](https://img.shields.io/badge/Language-Rust-orange)](https://www.rust-lang.org/)

---

OpenPRoT is a vendor-neutral, open-source firmware stack for Platform Root of Trust devices, built in Rust under the CHIPS Alliance. It implements MCTP, SPDM, and PLDM protocols for secure platform management.

## Architecture Overview

```mermaid
graph TD
    subgraph Services
        ATT["Attestation<br/>(SPDM)"]
        FWU["Firmware Update<br/>(PLDM)"]
        TEL["Telemetry"]
    end

    subgraph Protocols
        SPDM["SPDM<br/>Security & Auth"]
        PLDM["PLDM<br/>FW & Monitoring"]
        MCTP["MCTP<br/>Transport"]
    end

    subgraph Platform
        HAL["OpenPRoT HAL Traits<br/>(digest, ecdsa, cipher, mac, i2c, gpio)"]
        HUB["Hubris OS"]
        TOCK["Tock OS"]
        LIN["Linux"]
        RC["RustCrypto<br/>(software)"]
    end

    subgraph Hardware
        ASP["ASPEED AST1060<br/>ARM Cortex-M4F"]
        HACE["HACE Crypto<br/>Engine"]
        SPI["SPI / I2C / UART"]
    end

    ATT --> SPDM
    FWU --> PLDM
    TEL --> PLDM
    SPDM --> MCTP
    PLDM --> MCTP
    MCTP --> HAL
    HAL --> HUB
    HAL --> TOCK
    HAL --> LIN
    HAL --> RC
    HUB --> ASP
    ASP --> HACE
    ASP --> SPI

    style ATT fill:#509060,color:#fff
    style FWU fill:#509060,color:#fff
    style TEL fill:#509060,color:#fff
    style SPDM fill:#4a70a0,color:#fff
    style PLDM fill:#4a70a0,color:#fff
    style MCTP fill:#4a70a0,color:#fff
    style HAL fill:#a09825,color:#fff
    style HUB fill:#804080,color:#fff
    style TOCK fill:#804080,color:#fff
    style LIN fill:#804080,color:#fff
    style RC fill:#804080,color:#fff
    style ASP fill:#a05050,color:#fff
    style HACE fill:#a05050,color:#fff
    style SPI fill:#a05050,color:#fff
```

## Documentation

The full guide is organized into topic-based pages under [`docs/`](docs/README.md):

| Topic | Description |
|-------|-------------|
| [Introduction](docs/01-introduction.md) | What is OpenPRoT and why it exists |
| [Architecture](docs/02-architecture.md) | Layered architecture overview |
| [Protocols](docs/03-protocols/README.md) | MCTP, SPDM, and PLDM deep dives |
| [Repository Map](docs/04-repository-map.md) | All repos and how they relate |
| [Hardware Targets](docs/05-hardware-targets.md) | ASPEED chips and peripherals |
| [Getting Started](docs/06-getting-started.md) | Prerequisites and setup |
| [Building & Running](docs/07-building-and-running.md) | Build, QEMU, debug |
| [Tutorial: Secure Firmware Update](docs/08-tutorial-secure-firmware-update.md) | Step-by-step protocol walkthrough |
| [Services Deep Dive](docs/09-services-deep-dive.md) | Attestation, FW update, telemetry |
| [Standards & Compliance](docs/10-standards-and-compliance.md) | DMTF, TCG, NIST, OCP |
| [Use Cases](docs/11-use-cases.md) | Real-world scenarios |
| [Resources](docs/12-resources.md) | Links, references, license |

## Quick Start

```bash
# Install Rust and embedded target
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add thumbv7em-none-eabihf

# Clone and build
git clone https://github.com/OpenPRoT/openprot.git
cd openprot && cargo xtask build
```

See the [full getting started guide](docs/06-getting-started.md) for details.

---

*This guide was compiled from the [OpenPRoT GitHub organization](https://github.com/orgs/OpenPRoT/repositories) and related public sources.*
