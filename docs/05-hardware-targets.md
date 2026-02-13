# Hardware Targets

## ASPEED BMC Chips

OpenPRoT targets **ASPEED** ARM Cortex-M4 microcontrollers used as Platform Root of Trust controllers in server platforms:

| Chip | Core | Flash | RAM | Use Case |
|------|------|-------|-----|----------|
| **AST1060** | ARM Cortex-M4F | 1024 KB | 768 KB (640K + 128K non-cacheable) | Primary PRoT target |
| **AST1030** | ARM Cortex-M4F | — | — | QEMU-compatible alternative |

**Target triple:** `thumbv7em-none-eabihf` (ARM Cortex-M4 with hardware float)

**Toolchain:** Nightly Rust (2024-09-17 pinned in `aspeed-rust`)

## Memory Map (AST1060)

| Region | Address | Size | Purpose |
|--------|---------|------|---------|
| Flash | `0x80000000` | 1024 KB | Firmware storage |
| RAM | `0x00000000` | 640 KB | General purpose |
| RAM_NC | `0x000A0000` | 128 KB | Non-cacheable (DMA buffers, crypto contexts) |

In the Hubris configuration: 384 KB flash at `0x00000000`, 128 KB RAM at `0x60000`.

## Peripheral Drivers (`aspeed-rust`)

The `aspeed-rust` repository provides a comprehensive BSP with `embedded-hal` 1.0 trait implementations:

| Peripheral | Driver | embedded-hal Trait | Notes |
|-----------|--------|-------------------|-------|
| **I2C** | 14 buses (I2C0-I2C13) | `embedded_hal::i2c::I2c` | Master/slave, buffer mode, DMA mode, bus recovery |
| **SPI** | FMC + SPI controllers | `embedded_hal::spi::SpiBus<u8>` | NOR flash support (JESD216 modes), DMA, multi-CS |
| **SPI Monitor** | 4 instances (SPIM0-3) | — | Pass-through filter for host SPI bus monitoring |
| **UART** | 16550-compatible | `embedded_io::{Read, Write}` | Configurable baud/parity/stop, FIFO TX/RX |
| **GPIO** | Ports A through U (168 pins) | `embedded_hal::digital::{InputPin, OutputPin}` | Typestate pattern, interrupt support |
| **Watchdog** | 4 instances | `embedded_hal_old::watchdog` | Max ~4294 second timeout |
| **Timer** | Hardware timer | `embedded_hal_old::timer::CountDown` | One-shot and periodic modes |
| **Pin Control** | SCU registers | — | Pin mux configuration |
| **System Control** | Clock + reset | `ClockControl`, `ResetControl` traits | MCLK, YCLK, PCLK, HCLK management |

## Crypto Accelerators

The AST1060 includes hardware crypto engines accessed through the `aspeed-rust` HAL:

| Engine | Algorithms | Driver Features |
|--------|-----------|----------------|
| **HACE** (Hash and Crypto Engine) | SHA-1, SHA-224, SHA-256, SHA-384, SHA-512 | Hardware scatter-gather DMA, HMAC, scoped + owned API patterns |
| **RSA** (via Secure Boot Controller) | RSA modular exponentiation, PKCS#1 v1.5 | SBC SRAM at `0x79000000`, verify only (no key gen) |
| **ECDSA** | secp384r1 (P-384) | Hardware verification engine at `0x7e6f2000`, verify only |

The HACE engine uses a non-cacheable RAM section (`.ram_nc`) for DMA-safe hash contexts, supporting both:
- **Scoped API** — Reference-based, one-shot operations
- **Owned API** — Move-based, controller ownership flows through init→update→finalize chain

## PAC (Peripheral Access Crate)

Register-level access is provided by the external [`ast1060-pac`](https://github.com/AspeedTech-BMC/ast1060-pac) crate, generated from SVD files using `svd2rust`.

## QEMU Support

Both `aspeed-rust` and `hubris` support QEMU emulation:

```bash
qemu-system-arm -M ast1030-evb -nographic -kernel firmware.bin
```

The `ast1030-evb` QEMU machine is compatible with AST1060 firmware for development without hardware.

---

[Prev: Repository Map](04-repository-map.md) | [Next: Getting Started](06-getting-started.md)
