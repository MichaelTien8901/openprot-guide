---
layout: default
title: "13. Code Examples"
nav_order: 14
---

# Code Examples

Real code from the OpenPRoT repositories demonstrating key APIs and patterns.

## MCTP

### Linux MCTP Request ([`mctp-rs/mctp-linux/examples/mctp-req.rs`](https://github.com/OpenPRoT/mctp-rs/blob/main/mctp-linux/examples/mctp-req.rs))

Minimal synchronous MCTP request over Linux kernel sockets:

```rust
use mctp_linux::MctpLinuxReq;

fn main() -> std::io::Result<()> {
    let eid = 8;
    let mut ep = MctpLinuxReq::new(eid, None)?;

    let tx_buf = vec![0x02u8];
    ep.send(mctp::MCTP_TYPE_CONTROL, &tx_buf)?;

    let mut rx_buf = vec![0u8; 64];
    let (_msg_type, _ic, data) = ep.recv(&mut rx_buf)?;
    println!("response: {:02x?}", data);
    Ok(())
}
```

### Serial Echo Server ([`mctp-rs/standalone/examples/echo.rs`](https://github.com/OpenPRoT/mctp-rs/blob/main/standalone/examples/echo.rs))

MCTP echo server over serial transport, useful for QEMU testing:

```rust
use standalone::MctpSerialListener;

fn main() -> anyhow::Result<()> {
    let args: Vec<String> = std::env::args().collect();
    let port = &args[1];
    let eid = 9;

    let mut lis = MctpSerialListener::new(port, eid)?;
    loop {
        let mut buf = vec![0u8; 1024];
        let (msg_type, data) = lis.recv(&mut buf)?;
        lis.send(msg_type, data)?;
    }
}
```

Run with `socat` for a virtual serial link:
```bash
socat -d -d pty,rawer pty,rawer
# Then pass one PTY to the echo server and the other to a client
```

## Hubris Tasks

### Hello World with UART ([`hubris/task/helloworld/src/main.rs`](https://github.com/OpenPRoT/hubris/blob/master/task/helloworld/src/main.rs))

Minimal Hubris task demonstrating IPC and UART:

```rust
#![no_std]
#![no_main]

use userlib::*;
use embedded_io::{Read, Write};

task_slot!(UART, uart_driver);

#[export_name = "main"]
fn main() -> ! {
    let uart = drv_uart_api::Uart::from(UART.get_task_id());
    let _ = uart.write_all(b"Hello OpenPRoT!\r\n");

    // Echo loop
    let mut buf = [0u8; 1];
    loop {
        if uart.read(&mut buf).is_ok() {
            let _ = uart.write_all(&buf);
        }
    }
}
```

### HMAC via Digest Server ([`hubris/task/hmac-client/src/main.rs`](https://github.com/OpenPRoT/hubris/blob/master/task/hmac-client/src/main.rs))

Session-based HMAC through Hubris IPC to the digest server:

```rust
task_slot!(DIGEST, digest_server);

fn hmac_sha256(key: &[u8], data: &[u8]) -> [u8; 32] {
    let digest = drv_digest_api::Digest::from(DIGEST.get_task_id());
    let session = digest.open_hmac_session(
        drv_digest_api::DigestAlgorithm::Sha256,
        key,
    ).unwrap();

    digest.update_session(session, data).unwrap();
    let mut out = [0u8; 32];
    digest.finalize_session(session, &mut out).unwrap();
    out
}
```

### ECDSA Sign/Verify ([`hubris/task/ecdsa-test/src/main.rs`](https://github.com/OpenPRoT/hubris/blob/master/task/ecdsa-test/src/main.rs))

P-384 ECDSA via Hubris IPC:

```rust
task_slot!(ECDSA, ecdsa_server);

fn verify_signature(pubkey: &[u8; 97], message: &[u8], sig: &[u8; 96]) -> bool {
    let ecdsa = drv_ecdsa_api::OpenPRoTEcdsa::from(ECDSA.get_task_id());
    ecdsa.verify_p384(pubkey, message, sig).is_ok()
}
```

### I2C Master/Slave ([`hubris/task/i2c-client/src/main.rs`](https://github.com/OpenPRoT/hubris/blob/master/task/i2c-client/src/main.rs))

I2C communication through the Hubris I2C driver task:

```rust
task_slot!(I2C, mock_i2c);

fn i2c_write_read(addr: u8, write: &[u8], read_buf: &mut [u8]) {
    let i2c = drv_i2c_api::I2cDevice::from(I2C.get_task_id());
    i2c.write_read(addr, write, read_buf).unwrap();
}
```

## Hardware Drivers (aspeed-rust)

### SHA with HACE Hardware ([`aspeed-rust/src/tests/functional/hash_test.rs`](https://github.com/OpenPRoT/aspeed-rust/blob/main/src/tests/functional/hash_test.rs))

Using the ASPEED HACE hardware crypto engine:

```rust
use crate::crypto::hash::{DigestInit, DigestOp, HashAlgorithm};

fn sha256_hash(hace: &mut Hace, data: &[u8]) -> [u8; 32] {
    let mut ctx = hace.digest_init(HashAlgorithm::SHA256).unwrap();
    ctx.digest_update(data).unwrap();
    let mut out = [0u8; 32];
    ctx.digest_finalize(&mut out).unwrap();
    out
}
```

The HACE engine uses DMA from a non-cacheable RAM section (`.ram_nc`), supporting both scoped (reference-based) and owned (move-based) API patterns.

### ECDSA P-384 Verification ([`aspeed-rust/src/tests/functional/ecdsa_test.rs`](https://github.com/OpenPRoT/aspeed-rust/blob/main/src/tests/functional/ecdsa_test.rs))

Hardware-accelerated ECDSA verification with NIST test vectors:

```rust
use crate::crypto::ecdsa::{EcdsaVerify, EcdsaAlgorithm};

fn verify_p384(engine: &mut EcdsaEngine, pubkey: &[u8], hash: &[u8], sig: &[u8]) -> bool {
    engine.verify(EcdsaAlgorithm::P384, pubkey, hash, sig).is_ok()
}
```

The test suite includes 8 NIST P-384 test vectors covering both valid signatures and expected failures.

## PLDM Firmware Host ([`mctp-rs/pldm-file/examples/host.rs`](https://github.com/OpenPRoT/mctp-rs/blob/main/pldm-file/examples/host.rs))

A PLDM file host that serves firmware data and Platform Data Records (PDRs):

```rust
use pldm::PldmRequest;
use mctp::AsyncListener;

async fn serve_pldm(listener: &mut impl AsyncListener) {
    loop {
        let mut buf = vec![0u8; 4096];
        let (msg_type, data) = listener.recv(&mut buf).await.unwrap();

        let req = PldmRequest::decode(data).unwrap();
        let resp = handle_pldm_request(&req);
        listener.send(msg_type, &resp.encode()).await.unwrap();
    }
}
```

---

[Prev: Resources](12-resources.md) | [Next: Hubris Guide](14-hubris-guide.md)
