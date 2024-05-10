---
{"dg-publish":true,"permalink":"/00-notes/default-argument-promotion-in-c/","noteIcon":"","created":"2024-01-27T07:59:15.579+01:00","updated":"2024-01-27T07:59:32.671+01:00"}
---


## What the hell?

I am reading the world famous computer architecture book `Computer Systems: a programmer's perspective`, and following the open-source course `CSE351`. After finishing the first two chapters, I found it really hard to complete the parctice with no mistakes. Therefore, I have to `cheat` by writing some code to further understand how C code is translated into `x86-64` assembly code. I was writing a function to print the byte presentation of some integer values, which can be see in the following code block:

```c
#include <stdint.h>
#include <stdio.h>

void printByte(const char* ptr2Byte, int size) {
    while (size-- > 0) {
        printf("0x%02X\n", *(ptr2Byte++));
    }
}


int main()
{
    /* Char is by default signed */
    char num = 0x80;
    printf("Value stored in variable [num] = %d\n", num);
    
    printByte(&num, sizeof num);
    
    unsigned numMoved = (unsigned) num;
    printf("Value stored in variable [numMoved] = %u\n", numMoved);
    
    printByte((char*)&numMoved, sizeof numMoved);

    return 0;
}
```
> NOTE: `show_bytes` function is given in the chapter 2, with slightly different function signature, mainly the first argument is defined as `unsigned char`.

I am a bit suprised by the values printed in the console:

```bash
Value stored in variable [num] = -128
0xFFFFFF80
Value stored in variable [numMoved] = 4294967168
0xFFFFFF80
0xFFFFFFFF
0xFFFFFFFF
0xFFFFFFFF
```
In the function call of `printf("0x%02X\n", *(ptr2Byte++))`, the argument type is `char`, which is promoted to `int` by default (so callsed `default argument promotion in C`). Since the value assigned to the variable is `-128`, in which case `sign extension` makes sense. But why the specifier `02X` doesn't work here? I expected it to print `0x80` because of the width restriction imposed by the specifier.

It seems reasonable that `0x80` doesn't comply with the real value of `-128` represented by an 32-bit `int`, which is `0xFFFFFF80` instead of `0x80`. Therefore, I can make the assumption as follows:
- width specifier in the `printf` function doesn't always take effect unless it can make the representation of given argument without ambiguity. In the case the needed with exceeds the given one, argument is printed with its actual width. <sup>**[learned lesson 1]**</sup>

After checking the C documentation:

`If the value of the argument is negative, it results with the - flag specified and positive field width (Note: This is the minimum width: The value is never truncated.)`

In the meantime, I started understanding why the `unsigned char` is used in the `show_bytes` function provided in the book:
- it obviously makes the printing of byte representation get rid of the signedness issue, byte representation shouldn't have anything to do with the signedness. <sub>**[learned lesson 2]**</sub>

I am a bit shamed of myself not fully understanding the details of the demo implementation :(

