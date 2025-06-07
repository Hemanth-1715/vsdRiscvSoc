
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


## Add PATH 
```bash
export PATH=$HOME/riscv/bin:$PATH
```

## To make PATH addition permanent
```bash
nano ~/.bashrc
export PATH=$HOME/riscv/bin:$PATH
source ~/.bashrc
echo $PATH
```
- 1. First open the ~/.bashrc for editing the Shell config file
- 2. Add the export line at the end of the config file
- 3. Save and exit: (In nano, press Ctrl+O, then Enter to save. Then Ctrl+X to exit.) 
- 4. Apply the changes by using source 
- 5. Verify by running the echo and checking if it includes /home/yourusername/riscv/bin.

![Permanent PATH](<./Output Screenshots/Permanent PATH.png>)

## Verification Commands
```bash
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-gdb --version
riscv32-unknown-elf-objdump --version
```

![Add PATH](<./Output%20Screenshots/Add PATH and verification.png>)

- ✅ riscv32-unknown-elf-gcc --version → shows GCC 14.2.0 → Compiler is working
- ✅ riscv32-unknown-elf-objdump --version → shows Binutils 2.43.1 → Disassembler is working
- ✅ riscv32-unknown-elf-gdb --version → shows GDB 15.2 → Debugger is working

# Task 2: Compile “Hello, RISC-V”

## Objective
Compile a simple "Hello, RISC-V" program using the cross-compiler.

## C Program: helloworld.c
```bash
gedit helloworld.c
```
```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}

```
Save and close the .c file.
![Create](<./Output Screenshots/Creating Helloworld.c.png>)

## Compilation Command
```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -o helloworld.elf helloworld.c
```

### Explanation of Flags
| Flag                      | Meaning                                                       |
| ------------------------- | ------------------------------------------------------------- |
| `riscv32-unknown-elf-gcc` | RISC-V GCC compiler for 32-bit bare-metal targets (no OS)     |
| `-march=rv32imc`          | Target **RISC-V 32-bit** architecture with **IMC extensions** |
| `-mabi=ilp32`             | Use **ILP32** ABI (int, long, pointer = 32 bits)              |
| `helloworld.c`            | Your **C source file**                                        |
| `-o helloworld.elf`       | Name of the output **ELF executable**                         |

## ELF Verification
```bash
file helloworld.elf
```
The output confirms:

- The ELF is 32-bit
- It’s for the RISC-V architecture
- Uses the compressed instructions (RVC)
- Uses soft-float ABI(Application Binary Interface) (floating point handled in software)
- It’s statically linked

![Compile Output](<./Output Screenshots/Compiling Helloworld.c.png>)


# Task 3: From C to Assembly

## Objective
Generate the RISC-V assembly (`.s`) file from the C source and understand the prologue/epilogue.

## Command Used
```bash
riscv32-unknown-elf-gcc -S -O0 helloworld.c
```

This creates `hello.s` containing the assembly code in the same directory.
![Assembly Output](<./Output Screenshots/From .c to .s.png>)

## Contents of .s file
```bash
  	.file	"helloworld.c"
	.option nopic
	.attribute arch, "rv32i2p1_m2p0_a2p1_c2p0"
	.attribute unaligned_access, 0
	.attribute stack_align, 16
	.text
	.section	.rodata
	.align	2
.LC0:
	.string	"Hello, World!"
	.text
	.align	1
	.globl	main
	.type	main, @function
main:
	addi	sp,sp,-16
	sw	ra,12(sp)
	sw	s0,8(sp)
	addi	s0,sp,16
	lui	a5,%hi(.LC0)
	addi	a0,a5,%lo(.LC0)
	call	puts
	li	a5,0
	mv	a0,a5
	lw	ra,12(sp)
	lw	s0,8(sp)
	addi	sp,sp,16
	jr	ra
	.size	main, .-main
	.ident	"GCC: (g04696df096) 14.2.0"
	.section	.note.GNU-stack,"",@progbits
```

### What is the prologue and Epilogue?
The function prologue is the set of instructions at the beginning of a function that:
- Saves the return address (so the function can return properly),
- Saves any callee-saved registers it will use,
- Sets up the stack frame (allocates space on the stack),
- Prepares the environment for the function body.

In RISC-V assembly, a typical prologue includes instructions like:
```
    addi sp, sp, -16     # allocate stack space (adjust stack pointer)
    sw ra, 12(sp)        # save return address on stack
    sw s0, 8(sp)         # save frame pointer (s0) on stack
    addi s0, sp, 16      # set frame pointer to current stack pointer + offset
```
The function epilogue is the code at the end of the function that:
- Restores the saved registers,
- Restores the stack pointer to its original value,
- Returns control to the caller.
- Typical epilogue instructions:

```
lw ra, 12(sp)        # restore return address
lw s0, 8(sp)         # restore frame pointer
addi sp, sp, 16      # deallocate stack space (reset stack pointer)
ret                  # return to caller
```

# Task 4: Hex Dump & Disassembly

##  Objective
Convert the compiled ELF file into:
- A raw Intel HEX format file
- A human-readable disassembly using `objdump`

##  Commands Used

```bash
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex 
riscv32-unknown-elf-objdump -d hello.elf > hello.asm
```
The first command is used for creating a raw HEX from the ELF file
The second command is to disassemble the HEX file using objdump to a .asm file in the same directory
![O1](<./Output Screenshots/Cmd for raw HEX.png>)
![O2](<./Output Screenshots/Disassemble HEX.png>)

### Output .asm file created by disassembly by objdump
![.ASM](<./Output Screenshots/Assembly contents.png>)

## Breakdown of the instruction fields in main
| Column           | Meaning                                                     |
| ---------------- | ----------------------------------------------------------- |
| `0:`             | Offset address within the function or section               |
| `10162`          | Actual machine code in hex (the instruction in binary form) |
| `addi sp,sp,-16` | Human-readable disassembly of the machine code              |

Mnemonic
This is the operation name, like:
- addi (add immediate),
- lw (load word),
- sw (store word),
- jal (jump and link).
- It tells what the instruction does, just like function names in C.

Operands
These are the inputs and destination for the instruction:
- Registers (e.g., sp, a0, s0, etc.)
- Immediate values (constants, like -16)
- Addresses (for branches/jumps or memory access)

# Task 5: ABI and Register cheat sheet
## Objective
List all 32 RV32 integer registers with their:
- ABI Names - Application Binary Interface
- typical calling-convention roles

| Register | ABI Name | Description                          | Calling Convention Role |
|----------|----------|--------------------------------------|-------------------------|
| x0       | zero     | Hard-wired zero                      | Immutable constant zero |
| x1       | ra       | Return address                       | Caller-saved            |
| x2       | sp       | Stack pointer                        | Callee-saved            |
| x3       | gp       | Global pointer                       | Unallocatable           |
| x4       | tp       | Thread pointer                       | Unallocatable           |
| x5       | t0       | Temporary register 0                 | Caller-saved            |
| x6       | t1       | Temporary register 1                 | Caller-saved            |
| x7       | t2       | Temporary register 2                 | Caller-saved            |
| x8       | s0/fp    | Saved register 0 / Frame pointer     | Callee-saved            |
| x9       | s1       | Saved register 1                     | Callee-saved            |
| x10      | a0       | Function argument 0 / Return value 0 | Caller-saved            |
| x11      | a1       | Function argument 1 / Return value 1 | Caller-saved            |
| x12      | a2       | Function argument 2                  | Caller-saved            |
| x13      | a3       | Function argument 3                  | Caller-saved            |
| x14      | a4       | Function argument 4                  | Caller-saved            |
| x15      | a5       | Function argument 5                  | Caller-saved            |
| x16      | a6       | Function argument 6                  | Caller-saved            |
| x17      | a7       | Function argument 7                  | Caller-saved            |
| x18      | s2       | Saved register 2                     | Callee-saved            |
| x19      | s3       | Saved register 3                     | Callee-saved            |
| x20      | s4       | Saved register 4                     | Callee-saved            |
| x21      | s5       | Saved register 5                     | Callee-saved            |
| x22      | s6       | Saved register 6                     | Callee-saved            |
| x23      | s7       | Saved register 7                     | Callee-saved            |
| x24      | s8       | Saved register 8                     | Callee-saved            |
| x25      | s9       | Saved register 9                     | Callee-saved            |
| x26      | s10      | Saved register 10                    | Callee-saved            |
| x27      | s11      | Saved register 11                    | Callee-saved            |
| x28      | t3       | Temporary register 3                 | Caller-saved            |
| x29      | t4       | Temporary register 4                 | Caller-saved            |
| x30      | t5       | Temporary register 5                 | Caller-saved            |
| x31      | t6       | Temporary register 6                 | Caller-saved            |

# Task 6: Stepping with GDB 
## Objective
Debug a RISC-V ELF binary (`helloworld.elf`) using GDB, view the register contents, disassemble instructions, and step through the program using QEMU's GDB remote interface.

## Commands Used

### Step 1: Launch QEMU with GDB Server
```bash
qemu-riscv32 -g 1234 helloworld.elf
```
- Launches QEMU and halts execution.
- Enables a GDB server on port 1234, waiting for a debugger to connect.


| Component    | Explanation                                                             |
|--------------|-------------------------------------------------------------------------|
| qemu-riscv32 | This is the QEMU emulator for a generic 32-bit RISC-V CPU.              |
| -g 1234      | This tells QEMU to start in GDB debug mode and listen on TCP port 1234  |
| hello.elf    | This is the compiled RISC-V ELF binary                                  |

![Im1](<./Output Screenshots/Start.png>)

### Step 2: Launch GDB in another terminal
```bash
riscv32-unknown-elf-gdb helloworld.elf
```
- Starts the RISC-V version of GDB and loads the ELF with symbols.

### Step 3: Connect GDB to QEMU
```gdb
(gdb) target remote localhost:1234
```
- Establishes connection with QEMU.
![Im2](<./Output Screenshots/Starting QEMU.png>)

### Step 4: View all the avaiable functions
```gdb
(gdb) info functions
```
![Im3](<./Output Screenshots/Functions.png>)

### Step 5: Set Breakpoint at different functions (_start,main,exit) and Run
```gdb
(gdb) break _start
(gdb) continue
(gdb) break main
(gdb) continue
(gdb) break exit
(gdb) continue
```
- Execution halts at the corresponding function.

### Step 6: Step Through and Inspect
```gdb
(gdb) step
(gdb) info reg a0
(gdb) disas /r
```
- Step through the program.
- Inspect register contents (`a0`, `sp`, `ra`, etc.).
- Disassemble code to view human-readable assembly.

![Im4](<./Output Screenshots/Breakpoint for _start fn.png>)
![Im5](<./Output Screenshots/Breakpoint for main fn.png>)
![Im6](<./Output Screenshots/Breakpoint for exit fnn.png>)

---
## Learnings
- Cross-compilation for RISC-V
- ELF generation with correct architecture
- QEMU emulation
- Remote debugging with GDB
- Proper program execution flow
The "Hello, World!" appearing in Terminal 1 is the definitive proof that the RISC-V program executed successfully. 
 
 
The cross-compilation chain worked perfectly:
✅ C source compiled to RV32IMC ELF
✅ ELF loads and runs on RISC-V emulation
✅ printf() function works correctly
✅ Program terminates normally with exit code 0
✅ GDB can debug the RISC-V binary remotely