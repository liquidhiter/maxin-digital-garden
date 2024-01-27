---
{"dg-publish":true,"permalink":"/0-notes/gcc-s-enable-default-pie-option/","noteIcon":"","created":"2024-01-27T08:00:10.675+01:00","updated":"2024-01-27T17:54:37.244+01:00"}
---

## Error Encountered
`GCC` complanined `/usr/bin/ld: /tmp/cc2l29kd.o: relocation R_X86_64_32S against symbol g can not be used when making a PIE object; recompile with -fPIE`, when linking the following `x86_64` assembly code:
```assembly
	.text
	.globl	main
	.type	main, @function
main:

	movq	$0x1020304050607080, %rax  # put non-zero junk in bytes of %rax,
	movq	%rax, %rbx                 #   %rbx,
	movq	%rax, %rcx                 #   and %rcx

	movzbw	%al, %bx                   # pull lowest byte into %bx (16 bits) and zero-extend
                                       #   --> %rbx = 0x1020304050600080

	movzbq	%al, %rbx                  # pull lowest byte into %rbx and zero-extend
                                       #   --> %rbx = 0x80 even though MSB=1

	leaq	g, %rax                    # get address of global variable data
	movsbl	4(%rax), %ecx              # pull byte 0x80 into %ecx and sign-extend
                                       #   --> %rcx = 0xFFFFFF80 because 32-bit values
                                       #       placed in a x86-64 register zero out
                                       #       upper 32 bits
	ret
	.size	main, .-main               # . is the current location, .-main is the difference between here and the label main
	.globl	g
	.data
	.align 8
	.type	g, @object
	.size	g, 8
g:
	.quad	0x7778798081828384
```

## Solution
After searching the error message on Google, it was found that this issue was probably caused by `--enable-default-pie` option has been enabled in newer version of `GCC` by default<sup>[1]</sup>. The above assembly code was linked on a linux VM with the following `GCC` configurations:

```bash
# gcc --version
gcc (Debian 10.2.1-6) 10.2.1 20210110

# gcc -v
Configured with: ...... --enable-plugin --enable-default-pie --with-system-zlib ...
```

The issue can be solved by either of the following ways:
- using `clang` instead of `gcc`, guessing a similar `--enable-default-pie` option is not enabled by default
- disabling the `--enable-default-pie` option by adding `-no-pie` option, e.x. `gcc -Og -g -no-pie ...`
- using relative addressing<sup>[2]</sup>, i.e. `leaq g(%rip), %rax` instead of `leaq g, %rax`


## References
- [x] : https://nanxiao.me/en/gccs-enable-enable-default-pie-option-make-you-stuck-at-relocation-r_x86_64_32s-against-error/ ✅ 2024-01-27
- [x] : https://stackoverflow.com/questions/44967075/why-does-this-movss-instruction-use-rip-relative-addressing ✅ 2024-01-27