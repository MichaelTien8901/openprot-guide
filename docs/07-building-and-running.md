---
layout: default
title: "7. Building & Running"
nav_order: 8
---

# Building & Running

## 1. Building the Main Project

```bash
cd openprot

# Build everything
cargo xtask build

# Run tests
cargo xtask test

# Run all pre-checkin validation (build + test + clippy + fmt)
cargo xtask precheckin
```

### Available xtask Commands

```bash
cargo xtask build         # Build the project
cargo xtask test          # Run all tests
cargo xtask check         # Run cargo check
cargo xtask clippy        # Run clippy lints
cargo xtask fmt           # Format code
cargo xtask clean         # Clean artifacts
cargo xtask dist          # Build release distribution
cargo xtask docs          # Build mdbook documentation
cargo xtask header-check  # Check license headers
cargo xtask header-fix    # Fix missing license headers
cargo xtask precheckin    # Run all checks (CI equivalent)
```

## 2. Building the ASPEED Target

```bash
cd aspeed-rust

# Build for AST1060
cargo build --target thumbv7em-none-eabihf --release

# Generate a raw binary for flashing
cargo objcopy --target thumbv7em-none-eabihf --release -- -O binary ast10x0.bin

# Generate UART boot image
python3 scripts/gen_uart_booting_image.py ast10x0.bin
```

### Cargo Features

```bash
# Enable specific test suites
cargo build --features test-hash,test-hmac,test-ecdsa,test-rsa

# Enable SPI DMA and monitoring
cargo build --features spi_dma,spi_monitor

# Enable I2C target (slave) mode
cargo build --features i2c_target
```

## 3. Building Hubris App Images

Hubris uses `cargo xtask` with app TOML definitions:

```bash
cd hubris

# Build a complete firmware image
cargo xtask dist app/ast1060-starter/app.toml

# Build a single task for faster iteration
cargo xtask build app/ast1060-digest-test/app.toml digest_server

# Flash to hardware via probe-rs
cargo xtask flash app/ast1060-starter/app.toml
```

Available app images: `ast1060-starter`, `ast1060-i2c-scaffold`, `ast1060-digest-test`, `ast1060-ecdsa-test`, `ast1060-mctp-echo`.

## 4. Running in QEMU (Emulation)

### aspeed-rust

```bash
qemu-system-arm \
  -M ast1030-evb \
  -nographic \
  -kernel target/thumbv7em-none-eabihf/release/aspeed-ddk
```

### Hubris

```bash
# Build the image first
cargo xtask dist app/ast1060-i2c-scaffold/app.toml

# Run with GDB server enabled
qemu-system-arm \
  -M ast1030-evb \
  -nographic \
  -serial mon:stdio \
  -kernel target/ast1060-i2c-scaffold/dist/default/final.bin \
  -s -S

# In another terminal, connect GDB
gdb-multiarch
(gdb) target remote localhost:1234
(gdb) source target/ast1060-i2c-scaffold/dist/default/script.gdb
(gdb) continue
```

## 5. Debugging on Real Hardware

```bash
# Terminal 1: Start JLink GDB server
JLinkGDBServer -device cortex-m4 -if swd

# Terminal 2: Connect with GDB
gdb-multiarch target/thumbv7em-none-eabihf/release/aspeed-ddk
# In GDB:
#   target remote :2331
#   monitor semihosting IOClient 2
#   load
#   continue
```

## 6. Building Documentation

```bash
cd openprot
cargo xtask docs
# Opens in browser, or find HTML in docs/book/
```

Online docs are also available at: https://openprot.github.io/openprot/

## 7. Working with MCTP on Linux

```bash
cd mctp-rs

# Build all crates in the workspace
cargo build

# Run the PLDM firmware update CLI tool
cargo run -p pldm-fw-cli -- --help

# Query device firmware inventory
cargo run -p pldm-fw-cli -- inventory 0,8

# Update firmware from a package
cargo run -p pldm-fw-cli -- update 0,8 firmware.bin
```

---

[Prev: Getting Started](06-getting-started.md) | [Next: Tutorial: Secure Firmware Update](08-tutorial-secure-firmware-update.md)
