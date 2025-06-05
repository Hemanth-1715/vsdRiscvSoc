
# Task 2: Compile “Hello, RISC-V”

## Objective
Compile a simple "Hello, RISC-V" program using the cross-compiler.

## C Program: hello.c
```c
#include <stdio.h>

int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

## Compilation Command
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o hello.elf hello.c
```

### Explanation of Flags
- `riscv32-unknown-elf-gcc`: The RISC-V cross-compiler.
- `-march=rv32imc`: Specifies the target microarchitecture.
- `-mabi=ilp32`: Specifies the Application Binary Interface.
- `-o hello.elf`: Output ELF file.
- `hello.c`: Source file.

## ELF Verification
```bash
file hello.elf
```

## Output Screenshot
![Compile Output](./images/compile_hello_output.png)
