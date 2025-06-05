
# Task 3: From C to Assembly

## Objective
Generate the RISC-V assembly (`.s`) file from the C source and understand the prologue/epilogue.

## Command Used
```bash
riscv32-unknown-elf-gcc -S -O0 hello.c
```

This creates `hello.s` containing the assembly code.

## Sample Output (Prologue)
```asm
addi    sp,sp,-16
sw      ra,12(sp)
...
```

- `addi sp,sp,-16`: Allocates stack space.
- `sw ra,12(sp)`: Saves the return address.

## Output Screenshot
![Assembly Output](./images/c_to_assembly_output.png)
