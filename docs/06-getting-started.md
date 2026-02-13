# Getting Started

## Prerequisites

```bash
# 1. Install Rust (1.70+ required)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 2. Add the embedded ARM target
rustup target add thumbv7em-none-eabihf

# 3. Install cargo-binutils (for objcopy, etc.)
cargo install cargo-binutils
rustup component add llvm-tools-preview

# 4. (Optional) Install QEMU for emulation
sudo apt install qemu-system-arm    # Debian/Ubuntu
brew install qemu                    # macOS
```

## Clone the Repositories

```bash
# Main firmware stack
git clone https://github.com/OpenPRoT/openprot.git

# Individual protocol libraries (if working on them separately)
git clone https://github.com/OpenPRoT/mctp-rs.git
git clone https://github.com/OpenPRoT/spdm-lib.git
git clone https://github.com/OpenPRoT/pldm-lib.git

# Hardware support
git clone https://github.com/OpenPRoT/aspeed-rust.git
```

---

[Prev: Hardware Targets](05-hardware-targets.md) | [Next: Building & Running](07-building-and-running.md)
