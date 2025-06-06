
# Task 2: Compile “Hello, RISC-V”

## Objective
Compile a simple "Hello, RISC-V" program using the cross-compiler.

## C Program: hello.c
```bash
gedit helloworld.c
```
```c
volatile int *UART0 = (int *)0x10000000;

void _start() {

    const char *msg = "Hello World";

    while (*msg) {

        *UART0 = *msg++;

    }

    while (1); // Halt

}

```
Save and close the .c file.
### Why this .c file and not a general .c file with printf() statements?
- This is a bare metal code that directly writes characters to the memory-mapped UART (at address 0x10000000)
- Uses _start() as the entry point, not main()
- Does not use any standard C libraries
- Runs on hardware (or emulated hardware like QEMU) without an operating system
- However printf version: Needs a richer runtime, not suitable for truly bare-metal



![Create](<./Output Screenshots/Creating Helloworld.c.png>)

### Creating a linker script with .ld file type

```bash
ENTRY(_start)

MEMORY

{
  // This separates ROM (code/rodata) from RAM (data/bss), and avoids placing them all in one RWX segment.
  ROM (rx)  : ORIGIN = 0x80000000, LENGTH = 512K

  RAM (rwx) : ORIGIN = 0x80080000, LENGTH = 512K

} 
SECTIONS

{

  .text : {

    *(.text)

    *(.text*)

  } > ROM

  .rodata : {

    *(.rodata)

    *(.rodata*)

  } > ROM

  .data : {

    *(.data)

    *(.data*)

  } > RAM AT > ROM

  .bss : {

    *(.bss)

    *(.bss*)

    *(COMMON)

  } > RAM

  /DISCARD/ : {

    *(.comment)

    *(.note*)

  }

}
```
A linker script (.ld file) is a plain-text configuration file used by the linker (ld) to control how your program's code and data are laid out in memory.

Why Do We Need It?
In embedded systems (like RISC-V development), we don't have an operating system to manage memory. So we must tell the linker:
- Where the code (like _start, main) should go
- Where to place data, bss, and stack
- How to organize segments in Flash (ROM) and RAM
Without this, the program won't know where to load and run from.



## Compilation Command
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -nostartfiles -nostdlib -g -O0 -T linker.ld helloworld.c -o helloworld.elf
```

### Explanation of Flags
| Flag                      | Meaning                                                       |
| ------------------------- | ------------------------------------------------------------- |
| `riscv32-unknown-elf-gcc` | RISC-V GCC compiler for 32-bit bare-metal targets (no OS)     |
| `-march=rv32imc`          | Target **RISC-V 32-bit** architecture with **IMC extensions** |
| `-mabi=ilp32`             | Use **ILP32** ABI (int, long, pointer = 32 bits)              |
| `-nostartfiles`           | Do **not link** standard startup files (like `crt0.o`)        |
| `-nostdlib`               | Do **not link** standard libraries (`libc`, `libm`, etc.)     |
| `-g`                      | Include **debug symbols** for GDB debugging                   |
| `-O0`                     | Disable optimizations (**easier debugging**)                  |
| `-T linker.ld`            | Use custom **linker script** for memory layout                |
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
- It’s statically linked with debug info

![Compile Output](<./Output Screenshots/Compiling Helloworld.c.png>)
