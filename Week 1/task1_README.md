
# Task 1: Install & Sanity-Check the Toolchain

## Objective
Install the RISC-V GNU toolchain, add it to PATH, and verify the installation.

## Commands Used
```bash
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
```

![Unpacking and installing](./Output%20Screenshots/Unpacking and installing.png)

```bash
export PATH=$HOME/riscv/bin:$PATH
```

## Verification Commands
```bash
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-gdb --version
riscv32-unknown-elf-objdump --version
```
![Add PATH](./Output%20Screenshots/Add PATH and verification.png)



