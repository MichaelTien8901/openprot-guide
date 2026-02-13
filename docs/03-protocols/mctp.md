# MCTP — Transport Layer

**Management Component Transport Protocol** (DMTF DSP0236)

MCTP is the "postal service" for platform management — it delivers messages between components regardless of the physical connection.

## Key Concepts

- **EID (Endpoint ID)** — Address for each component on the MCTP network
- **Bindings** — Physical transport adapters (I2C/SMBus, serial, USB)
- **Message Types** — Identifies which protocol a message belongs to (SPDM, PLDM, NVMe, CXL, etc.)
- **Tags** — Track request/response pairs; owned by the initiator
- **Fragmentation** — Large messages are split into MTU-sized packets and reassembled

## Repositories

### `mctp-rs` — Comprehensive Workspace (by Code Construct)

The primary MCTP implementation. A multi-crate workspace (MIT/Apache-2.0) providing both Linux and embedded support:

| Crate | Target | Purpose |
|-------|--------|---------|
| `mctp` | `no_std` | Base types and traits (`Eid`, `Tag`, `ReqChannel`, `Listener`, async variants) |
| `mctp-linux` | `std` | Linux kernel MCTP socket transport (AF_MCTP, requires Linux 5.15+) |
| `mctp-estack` | `no_std` | Full embedded stack: fragmentation, reassembly, routing, timeout management |
| `mctp-usb-embassy` | `no_std` | USB transport for Embassy-based embedded systems |
| `standalone` | `std` | Serial transport over Linux TTY (for QEMU testing) |
| `pldm` | `no_std` | PLDM base protocol types (uses `deku` for serialization) |
| `pldm-fw` | `no_std`/`std` | PLDM firmware update: FD responder (`no_std`) + UA requester (`std`) |
| `pldm-fw-cli` | `std` | CLI tool for firmware inventory, update, cancel, package inspection |

The embedded stack (`mctp-estack`) is fully configurable at compile time via environment variables:

| Variable | Default | Purpose |
|----------|---------|---------|
| `MCTP_ESTACK_MAX_MESSAGE` | 1032 | Maximum message payload size |
| `MCTP_ESTACK_NUM_RECEIVE` | 4 | Concurrent reassembly slots |
| `MCTP_ESTACK_FLOWS` | 64 | Flow tracking table size |
| `MCTP_ESTACK_MAX_MTU` | 255 | Maximum transmission unit |

### `mctp-lib` — Thin Embedded Wrapper

A wrapper around `mctp-estack` that adds a `Router` with a cookie-based handle API:

- **Listener handles** — Register for a message type, receive incoming requests
- **Request handles** — Send to a destination EID, receive responses
- **`Sender` trait** — Pluggable transport binding abstraction

## Transport Bindings

| Binding | Crate | Notes |
|---------|-------|-------|
| **I2C/SMBus** | `mctp-estack` | Uses `smbus-pec` for packet error checking |
| **Serial** | `mctp-estack` | CRC-based framing for serial links |
| **USB** | `mctp-usb-embassy` | Embassy-based USB transport |
| **Linux sockets** | `mctp-linux` | Kernel AF_MCTP sockets (Linux 5.15+) |

## Hubris Integration

In the Hubris fork, MCTP runs as an isolated `mctp-server` task implementing a router with serial transport binding. The `mctp-echo` task demonstrates basic message exchange.

## Example: Sending an MCTP Message (Linux)

```rust
use mctp_linux::MctpLinuxReq;

// Connect to endpoint with EID 8
let mut ep = MctpLinuxReq::new(8, None)?;

// Send a control message
let tx_buf = vec![0x02u8];
ep.send(mctp::MCTP_TYPE_CONTROL, &tx_buf)?;

// Receive response
let mut rx_buf = vec![0u8; 64];
let (msg_type, integrity_check, data) = ep.recv(&mut rx_buf)?;
```

---

[Prev: Protocols Overview](README.md) | [Next: SPDM](spdm.md)
