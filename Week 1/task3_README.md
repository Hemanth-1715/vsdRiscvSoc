
# Task 3: From C to Assembly

## Objective
Generate the RISC-V assembly (`.s`) file from the C source and understand the prologue/epilogue.

## Command Used
```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -S helloworld.c
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
	.string	"Hello World"
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
	call	printf
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