
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

- ✅ C source compiled to RV32IMC ELF
- ✅ ELF loads and runs on RISC-V emulation
- ✅ printf() function works correctly
- ✅ Program terminates normally with exit code 0
- ✅ GDB can debug the RISC-V binary remotely

# Task 7: Running Under an Emulator (QEMU)

## Objective
Execute a bare-metal RISC-V ELF binary using emulators (Spike and QEMU) to simulate real hardware execution and observe UART console output in a production-like environment.


## Creating a bare-metal .c file to print "Hello World"
```c
// Working Hello World bare-metal program 
#define UART_BASE 0x10000000
void _start(void) {
    // Initialize stack pointer - this is crucial!
    asm volatile ("li sp, 0x80400000");
    // Direct UART access - no function calls

    volatile unsigned char *uart = (volatile unsigned char *)UART_BASE;
    // Print "Hello, World!\n"
    char *msg1 = "Hello, World!\n";

    while (*msg1) {

        *uart = *msg1;

        msg1++;

    }
    // Print "This is bare-metal RISC-V!\n"

    char *msg2 = "This is bare-metal RISC-V!\n";

    while (*msg2) {

        *uart = *msg2;

        msg2++;

    }

    // Halt execution with infinite loop

    while (1) {
        asm volatile ("wfi");  // Wait for interrupt (saves power)
    }
}
```
## Creating a linker script - .ld

```c
ENTRY(_start)
MEMORY {

    RAM : ORIGIN = 0x80200000, LENGTH = 126M

}
SECTIONS {
    .text : {

        *(.text)

        *(.text.*)

    } > RAM
    .data : {

        *(.data)

        *(.data.*)
    } > RAM
    .bss : {

        *(.bss)

        *(.bss.*)

    } > RAM
}
```
## Why the above approach
A linker script (.ld file) is a plain-text configuration file used by the linker (ld) to control how your program's code and data are laid out in memory.

Why Do We Need It?
In embedded systems (like RISC-V development), we don't have an operating system to manage memory. So we must tell the linker:
- Where the code (like _start, main) should go
- Where to place data, bss, and stack
- How to organize segments in Flash (ROM) and RAM
- Without this, the program won't know where to load and run from.


### Why this .c file and not a general .c file with printf() statements?
- This is a bare metal code that directly writes characters to the memory-mapped UART (at address 0x10000000)
- Uses _start() as the entry point, not main()
- Does not use any standard C libraries
- Runs on hardware (or emulated hardware like QEMU) without an operating system
- However printf version: Needs a richer runtime, not suitable for truly bare-metal

### Why This Approach Matters:
- **Real-world relevance**: Mirrors actual embedded development workflow
- **Hardware understanding**: Demonstrates system-level programming skills
- **Testing methodology**: Shows production verification techniques
- **Progressive learning**: Builds upon previous debugging experience

---
## Commands Used

### Using QEMU System Emulator
```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -nostdlib -nostartfiles -ffreestanding -T bm.ld -o hello_bm.elf hello_bm.c //for compiling and getting .elf file
qemu-system-riscv32 -nographic -machine virt -kernel hello_bm.elf

```
- `qemu-system-riscv32`: Full system emulation for RISC-V 32-bit
- `-nographic`: Disables graphical output, redirects console to terminal
- `-kernel hello_bm.elf`: Loads ELF as kernel image for bare-metal execution

---
![C1](<./Output Screenshots/Compiling baremetal.png>)
![Emu1](<./Output Screenshots/QEMU_Emulation(1).png>)
![Emu2](<./Output Screenshots/QEMU_Emulation(2).png>)


## Key Learnings and Comparison

| Aspect              | Task 6 (GDB)           | Task 7 (Emulator)        |
|---------------------|------------------------|--------------------------|
| **Purpose**         | Debugging & Analysis   | Execution & Testing      |
| **Control**         | Interactive stepping   | Continuous execution     |
| **Visibility**      | Instruction-level      | Output-level only        |
| **Environment**     | Debug simulator        | Hardware simulation      |
| **Use Case**        | "What's happening?"    | "Does it work?"          |

# Task 8: Exploring GCC Optimisation 

## Objective
Observe the differences that appear in the .s files by using two types of gcc optimisation

## Commands Used
```bash
riscv32-unknown-elf-gcc -S -O0 helloworld.c -o heloworld_1.s
riscv32-unknown-elf-gcc -S -O2 helloworld.c -o heloworld_2.s
```

## Output .s files of -O0 and -O2 optimisation (LHS : -O0; RHS : -O2)
![Comp](<./Output Screenshots/Comparison.jpg>)

## Explanation of Differences

| Aspect                    | `-O0`                                  | `-O2`                                   |
| ------------------------- | -------------------------------------- | --------------------------------------- |
| **Prologue/Epilogue**     | Creates stack frame, saves/restores RA | No stack frame needed                   |
| **Temporary Variables**   | Stored in memory                       | Eliminated                              |
| **Loads/Stores**          | Multiple unnecessary memory accesses   | Removed via register usage              |
| **Function Overhead**     | Conservative; preserves state          | Aggressive optimization; minimal code   |
| **Dead Code Elimination** | Not done                               | Performed (eliminates unused variables) |

### Why these differences?
-O0 prioritizes debuggability:
  - Keeps variables in memory
  - Avoids instruction reordering
  - Easier to inspect with a debugger


-O2 focuses on execution speed and code size:
  - Eliminates redundant loads/stores
  - Removes unnecessary variables and stack usage
  - Performs constant folding, inlining, strength reduction, etc.

# Task 9: Inline Assembly Basics 

## Objective
Implement a C function to read the RISC-V cycle counter using inline assembly and provide detailed explanation of assembly constraints.

## Commands Used
```bash
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -nostdlib -nostartfiles -T linker.ld -o inline.elf start.S inline.c
qemu-system-riscv32 -nographic -machine virt -kernel inline.elf
```

## A function that returns the cycle counter by reading CSR 0xC00 using inline asm
```c
static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, 0xC00" : "=r"(c));
    return c;
}
```
## A bare metal .c to print the cycle counter value 

```c

// rdcycle_std.c - RISC-V cycle counter with standard headers
#include <stdint.h>

#define UART_BASE 0x10000000

// RISC-V cycle counter function using CSR 0xC00
static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, 0xC00" : "=r"(c));
    return c;
}

// Simple UART output function
void uart_putc(char c) {
    volatile uint8_t *uart = (volatile uint8_t *)UART_BASE;
    *uart = c;
}

void uart_puts(const char *str) {
    while (*str) {
        uart_putc(*str);
        str++;
    }
}

void uart_puthex(uint32_t value) {
    char hex[] = "0123456789ABCDEF";
    uart_puts("0x");
    for (int i = 28; i >= 0; i -= 4) {
        uart_putc(hex[(value >> i) & 0xF]);
    }
}

int main(void) {
    uart_puts("Testing cycle counter...\n");
    
    // Read cycle counter using CSR 0xC00
    uint32_t cycle_value = rdcycle();
    
    uart_puts("Cycle read successful!\n");
    uart_puts("Cycle value: ");
    uart_puthex(cycle_value);
    uart_putc('\n');
    
    
    return 0;
}
```

## A startup file .S 
Since the linker expects an entry point symbol _start (by default), we write a minimal assembly start that sets up the stack pointer and calls main.
```c
    .section .text
    .globl _start
_start:
    la sp, _stack_top        # Set stack pointer to top of stack
    call main                # Call main()
    # If main returns, loop forever
1:
    wfi
    j 1b

    .section .bss
    .space 4096              # reserve 4KB stack space (adjust as needed)
_stack_top:

```
## Commands Used
```bash
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -nostdlib -nostartfiles -T linker.ld -o inline.elf start.S inline.c
qemu-system-riscv32 -machine virt -nographic -kernel inline.elf
```
![counter](<./Output Screenshots/Cycle counter.png>)

## Explanation of Components
1. asm volatile(...)
    - This tells the compiler to insert inline assembly into the C code.
    - asm: keyword for GCC-style inline assembly.
    - volatile: tells the compiler not to optimize away or reorder this assembly instruction.
    - Without volatile, the compiler might remove or move this line if it thinks it has no side effects (since the output c is not guaranteed to be used immediately).


2. "csrr %0, cycle"
    - This is the RISC-V assembly instruction that reads the cycle CSR (0xC00).
    - csrr: Control and Status Register Read instruction
    - %0: placeholder for the first output operand (in this case, c)
    - It tells the assembler: “substitute this with a register that will hold the output.”


3. "=r"(c)
    - his is the output operand constraint, which tells the compiler:
    - =: this is write-only (output-only). The value of c is written by the instruction.
    - r: use a general-purpose register to hold the value of cycle.
    - (c): bind this output to the C variable c.


# Task 10:  Memory-Mapped I/O Demo
## Objective
Toggle a GPIO register located at address 0x10012000 using bare-metal C, and prevent the compiler from optimizing away the memory access.

## Commands Used
```bash
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -nostdlib -nostartfiles -T linker.ld -o tog.elf start.S tog.c
qemu-system-riscv32 -machine virt -nographic -kernel tog.elf
```

## Bare-metal toggle program
```c
#include <stdint.h>
#define UART_BASE 0x10000000
#define UART_THR  (*(volatile uint8_t *)(UART_BASE + 0x00))  // UART transmit register
#define GPIO_ADDR 0x10012000

void uart_putc(char c) {
    UART_THR = c;
}

void uart_puts(const char *s) {
    while (*s) uart_putc(*s++);
}

void delay() {
    for (volatile int i = 0; i < 100000; ++i);
}

int main() {
    volatile uint32_t *gpio = (volatile uint32_t *)GPIO_ADDR;

    uart_puts("GPIO set HIGH\n");
    *gpio = 0x1;        // Set GPIO high
    delay();

    uart_puts("GPIO set LOW\n");
    *gpio = 0x0;        // Set GPIO low

    // Halt the processor using `wfi` (wait for interrupt)
    while (1) {
        __asm__ volatile ("wfi");
    }
}

```

## Startup and linker file
```bash
    .section .text
    .globl _start
_start:
    la sp, _stack_top        # Set stack pointer to top of stack
    call main                # Call main()
    # If main returns, loop forever
1:
    wfi
    j 1b

    .section .bss
    .space 4096              # reserve 4KB stack space (adjust as needed)
_stack_top:
```


```bash
ENTRY(_start)

MEMORY {
    FLASH : ORIGIN = 0x80200000, LENGTH = 4M   /* For code, read+execute */
    RAM   : ORIGIN = 0x80600000, LENGTH = 12M  /* For data, read+write */
}

SECTIONS {
    .text : {
        *(.text*)
    } > FLASH

    .rodata : {
        *(.rodata*)
    } > FLASH

    .data : {
        *(.data*)
    } > RAM

    .bss : {
        *(.bss*)
        *(COMMON)
    } > RAM
}

```

![GPIO](<./Output Screenshots/GPIO.png>)

## Key Concepts
### volatile Keyword
- Tells the compiler not to optimize access to the memory location.
- Ensures that every read/write to *gpio actually occurs, even if it looks redundant in the code.
- Without volatile, the compiler might optimize out writes that appear to have no side effects, which is dangerous in embedded I/O.

### Memory Alignment
- uint32_t is 4 bytes, so 0x10012000 must be 4-byte aligned — which it is.
- Misaligned accesses can cause hardware faults or undefined behavior on some systems.

# Task 11: Linker Script 101
## Objective
To create a  minimal linker script that places .text at 0x00000000 and .data at 0x10000000 for RV32IMC.

## Linker Script
```bash
ENTRY(_start)

SECTIONS {
    .text 0x00000000 : {
        *(.text)
        *(.text.*)
    }

    .data 0x10000000 : {
        *(.data)
        *(.data.*)
    }
}

```
## Explanation: Flash vs SRAM
1. Flash (e.g., address 0x00000000):
    - Non-volatile storage.
    - Stores code and sometimes constants.
    - Read-only during runtime unless special controller access is used.


2. SRAM (e.g., address 0x10000000):
    - Volatile memory.
    - Used for read/write data sections like .data and .bss.
    - Loses content on power-off.


## Reason for separation:
- At boot, code runs from Flash.
- Global/static variables need to be writable — placed in SRAM.
- The bootloader or startup code often copies .data from Flash to SRAM before jumping to main().

# Task 12: Start-up Code & crt0 

## Objective
To understand crt0.S role in a bare-metal RISC-V program and finding out where to get one.

## Startup script
```asm
    .section .text
    .globl _start
_start:
    # Set up the stack pointer
    la sp, _stack_top

    # Zero out the .bss section
    la a0, __bss_start
    la a1, __bss_end
zero_bss:
    bgeu a0, a1, bss_done
    sw zero, 0(a0)
    addi a0, a0, 4
    j zero_bss
bss_done:

    # Call main()
    call main

    # Infinite loop if main returns
1:
    wfi
    j 1b

    .section .bss
    .globl __bss_start
    .globl __bss_end
__bss_start:
    .space 0x1000    # Adjust size as needed
__bss_end:

    .space 4096
_stack_top:
```

## Where to get a crt0.S?
1. Newlib: A popular C library for embedded systems, includes a generic crt0.S that you can adapt.


    Examples in SDKs:
    - Microchip’s SoftConsole or SiFive Freedom E SDK.
    - PlatformIO or Zephyr RTOS projects.

2. VSDSquad or GitHub: Many educational projects and RISC-V demos on GitHub (search for riscv crt0.S).


