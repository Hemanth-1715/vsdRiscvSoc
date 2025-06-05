
# Task 1: Install & Sanity-Check the Toolchain

## Objective
Install the RISC-V GNU toolchain, add it to PATH, and verify the installation.

## Commands Used
```bash
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
```

![Unpacking and installing](<./Output Screenshots/Unpacking and installing.png>)


## Commands to inspect and view all the files inside the bin folder (binary files)
```bash
cd opt
cd riscv
ls bin
```
![Inspect](<./Output Screenshots/Inspecting files.png>)


## Add PATH and Verification Commands
```bash
export PATH=$HOME/riscv/bin:$PATH
```
```bash
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-gdb --version
riscv32-unknown-elf-objdump --version
```

![Add PATH](<./Output%20Screenshots/Add PATH and verification.png>)

✅ riscv32-unknown-elf-gcc --version → shows GCC 14.2.0 → Compiler is working
✅ riscv32-unknown-elf-objdump --version → shows Binutils 2.43.1 → Disassembler is working
✅ riscv32-unknown-elf-gdb --version → shows GDB 15.2 → Debugger is working


