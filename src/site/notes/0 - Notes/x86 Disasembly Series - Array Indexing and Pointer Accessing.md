---
{"dg-publish":true,"permalink":"/0-notes/x86-disasembly-series-array-indexing-and-pointer-accessing/","noteIcon":"","created":"2024-01-27T08:14:38.006+01:00","updated":"2024-01-27T08:16:04.348+01:00"}
---

## Array Indexing and Pointer Accessing

### Introduction
The `C` syntax supports the following ways of defining arrays:
```c
int array[3] = {1, 2, 3};
int *ptr2Int = malloc(sizeof(int) * 10);
```
As we know, `array` is interchangeable with the pointer in some cases. But what is the difference of accessing the elements in the array via the index and the pointer?
```c
array[0] = -1;
*ptr2Int = -1;
```

### Experiments
#### Environment
- OS: `Debian GNU/Linux 11 (bullseye) x86_64`
- GCC: `gcc (Debian 10.2.1-6) 10.2.1 20210110`

#### Source Code
```c
/* ************************************************************************
> File Name:     array.c
> Author:        Maxin
> mail:          lyingflatlearner@gmail.com
> Created Time:  Sun Dec 31 11:05:32 2023
> Description:   
 ************************************************************************/
// #include <stdlib.h>

int main(int argc, char** argv) {
		int a[2];
		a[0] = 1;
		a[1] = 2;

		int *ptr2Int = malloc(sizeof(int) * 2);
		*ptr2Int++ = 1;
		*ptr2Int = 2;

		return 0;
}
```

#### Assembly Code
##### Optimization level `O0`
```assembly
.file	"array.c"
	.text
	.globl	main
	.type	main, @function
main:
.LFB0:
	.cfi_startproc
  # 8B (return address), implicitly sub $8, %rsp
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
  # Save stack pointer address to %rbp
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6

  # 4B (argc) + 8B (argv) + 4B (a) + 8B (2 integer elements) + 8B (ptr2Int)
	subq	$32, %rsp

  # argument argc
	movl	%edi, -20(%rbp)
  
  # argument argv
	movq	%rsi, -32(%rbp)

	# assign the values via indexes
	movl	$1, -16(%rbp)
	movl	$2, -12(%rbp)

	movl	$8, %edi
	call	malloc@PLT

  # Save the address returned by malloc function into ptr2Int
	movq	%rax, -8(%rbp)
  # Copy it back ? WHY? is rax being used between these two instructions?
	movq	-8(%rbp), %rax
  # Points to the next element (ptr2Int[1] / ptr2Int++)
	leaq	4(%rax), %rdx
  # Save the address of next element into ptr2Int
	movq	%rdx, -8(%rbp)

  # Assign one to the first element, ptr2Int[0] / *ptr2Int
	movl	$1, (%rax)
  # Copy the address of the second element into rax
	movq	-8(%rbp), %rax
	movl	$2, (%rax)
	
  movl	$0, %eax
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	main, .-main
	.ident	"GCC: (Debian 10.2.1-6) 10.2.1 20210110"
	.section	.note.GNU-stack,"",@progbits
```
##### array indexing
```assembly
	movl	$1, -16(%rbp)
	movl	$2, -12(%rbp)
```
- `M[%rbp - 16] = 1` where `rbp` stores the address of the stack pointer

##### pointer access
```assembly
  # Save the address returned by malloc function into ptr2Int
	movq	%rax, -8(%rbp)
  # Copy it back ? Is rax being used between these two instructions?
	movq	-8(%rbp), %rax
  # Points to the next element (ptr2Int[1] / ptr2Int++)
	leaq	4(%rax), %rdx
  # Save the address of next element into ptr2Int
	movq	%rdx, -8(%rbp)

  # Assign one to the first element, ptr2Int[0] / *ptr2Int
	movl	$1, (%rax)
  # Copy the address of the second element into rax
	movq	-8(%rbp), %rax
	movl	$2, (%rax)
```
- the logic implied in the above assembly code is different than the one in the code
- `ptr2Int` is incremented (note: this is the pointer arithmetic) before it is needed
```assembly
## following the source code logic
 
  # Save the address returned by malloc function into ptr2Int
	movq	%rax, -8(%rbp)
  # Copy it back ? Is rax being used between these two instructions?
	movq	-8(%rbp), %rax

  # Assign one to the first element, ptr2Int[0] / *ptr2Int
	movl $1, (%rax)
  # Pointer increment
  # Points to the next element (ptr2Int[1] / ptr2Int++)
	leaq	4(%rax), %rax
  # Save the address of next element into ptr2Int
	movq	%rax, -8(%rbp)
  # Assign two to the second element, ptr2Int[1]
	movl $2, (%rax)
```
