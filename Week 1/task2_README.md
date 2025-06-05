
# Task 2: Compile “Hello, RISC-V”

## Objective
Compile a simple "Hello, RISC-V" program using the cross-compiler.

## C Program: hello.c
```c
#include <stdio.h>

int main() {
    printf("Hello World");
    return 0;
}
```

## Compilation Command
```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -o helloworld.elf helloworld.c
```

### Explanation of Flags
- `riscv32-unknown-elf-gcc`: The RISC-V cross-compiler.
- `-march=rv32imac`: Specifies the target microarchitecture.
- `-mabi=ilp32`: Specifies the Application Binary Interface.
- `-o hello.elf`: Output ELF file.
- `helloworld.c`: Source file.

## ELF Verification
```bash
file hello.elf
```

## Output Screenshot
![Compile Output](./images/compile_hello_output.png)
