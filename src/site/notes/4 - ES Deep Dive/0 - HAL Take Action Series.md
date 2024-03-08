---
{"dg-publish":true,"permalink":"/4-es-deep-dive/0-hal-take-action-series/","noteIcon":"","created":"2024-02-26T20:34:12.366+01:00","updated":"2024-03-07T23:16:52.408+01:00"}
---

## 零碎知识点
### flash烧写
- 如何实现数据的写入？
	- `API` 操纵硬件？
- 不同的通信接口？
	- UART
	- USB
### keil 默认编译器参数
```c
--c99 -c --cpu Cortex-M3 -g -O3 --apcs=interwork --split_sections -I ../Core/Inc -I ../Drivers/STM32F1xx_HAL_Driver/Inc -I ../Drivers/STM32F1xx_HAL_Driver/Inc/Legacy -I ../Drivers/CMSIS/Device/ST/STM32F1xx/Include -I ../Drivers/CMSIS/Include
-I./RTE/_0301_led
-IC:/Users/maxin/AppData/Local/Arm/Packs/ARM/CMSIS/6.0.0/CMSIS/Core/Include
-IC:/Users/maxin/AppData/Local/Arm/Packs/Keil/STM32F1xx_DFP/2.4.1/Device/Include
-D__UVISION_VERSION="539" -D_RTE_ -DSTM32F10X_MD -D_RTE_ -DUSE_HAL_DRIVER -DSTM32F103xB
-o 0301_led\*.o --omf_browse 0301_led\*.crf --depend 0301_led\*.d
```
- `--cpu`: 处理器型号
- `-DSTM32F10X_MD`: device?
- ` -D_RTE_`: what?
- ` -DUSE_HAL_DRIVER`: use HAL 
- ` -DSTM32F103x`: board?
- `--omf_browse`: 
	- Enables the generation and storing of source browser information
	- what ?

### 待解决的问题
- `Reset：复位，让程序从任一状态变为初始状态（跳到Reset Handler）；`
	- `reset handler`
```c
; Reset handler
Reset_Handler    PROC
                 EXPORT  Reset_Handler             [WEAK]
     IMPORT  __main
     IMPORT  SystemInit
                 LDR     R0, =SystemInit
                 BLX     R0
                 LDR     R0, =__main
                 BX      R0
                 ENDP
```
- `[WEAK]` : `Reset_Handler` can be defined somewhere else
- `SystemInit`: a function defined in `system_stm32f1xx.c` file
```c
/**
  * @brief  Setup the microcontroller system
  *         Initialize the Embedded Flash Interface, the PLL and update the 
  *         SystemCoreClock variable.
  * @note   This function should be used only after reset.
  * @param  None
  * @retval None
  */
void SystemInit (void)
{
#if defined(STM32F100xE) || defined(STM32F101xE) || defined(STM32F101xG) || defined(STM32F103xE) || defined(STM32F103xG)
  #ifdef DATA_IN_ExtSRAM
    SystemInit_ExtMemCtl(); 
  #endif /* DATA_IN_ExtSRAM */
#endif 

  /* Configure the Vector Table location -------------------------------------*/
#if defined(USER_VECT_TAB_ADDRESS)
  SCB->VTOR = VECT_TAB_BASE_ADDRESS | VECT_TAB_OFFSET; /* Vector Table Relocation in Internal SRAM. */
#endif /* USER_VECT_TAB_ADDRESS */
}
```
- does the order of `__main` and `SystemInit` matter ?


- CPU direct access to NorFlash
- CPU can't directly access to
	- SPI Flash
	- SD Card
	- USB Disk
		- on-chip controllers
			- 存储介质访问负责的时候都需要有控制器
	- these are not `XIP`
![Z - assets/images/Pasted image 20240227071907.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240227071907.png)
- 变量根据其读写性质被存储在不同的存储介质中
	- RAM中存放可读可写的变量
	- ROM中只能存放可读的变量
![Z - assets/images/Pasted image 20240227072558.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240227072558.png)
### GPIO
![Z - assets/images/Pasted image 20240227205433.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240227205433.png)
- 电源和时钟使能
	- 低功耗考虑
- I/O pin复用
	- 模式mode选择
- 输入/输出选择
- 对应数据的不同

### 寄存器
- 读写寄存器
	- 不能影响其它位
```c
// first, read the value from the register
val = HWREG_READ(reg_addr);
val = val | 0x1;
HWREG_WRITE(reg_addr) = val;
```
> 读取 -> 修改 -> 写回

- 额外的置位和清零寄存器
	- 相应的bit影响实际寄存器的数值
	- 需要芯片支持 `set-and-clear protocol`

### LED Blink
> main.c
```c
#define RCC_APB2ENR		(0x40021000 + 0x18)
#define GPIOC_BASE		(0x40011000)
#define GPIOC_CRH			(GPIOC_BASE + 0x04)
#define GPIOC_ODR     (GPIOC_BASE + 0X0C)

/* Function Prototype */
void delay(volatile int i);

void delay(volatile int i)
{
	while (i--);
}

int main(void)
{
	volatile unsigned int *pRccApb2Enr;
	volatile unsigned int *pGpiocCrH;
	volatile unsigned int *pGpiocOdr;
	/* Enable the clock */
	pRccApb2Enr = (volatile unsigned int *)RCC_APB2ENR;
	/* Set the GPIO mode */
	pGpiocCrH   = (volatile unsigned int *)GPIOC_CRH;
	/* Write value to the GPIO register */
	pGpiocOdr   = (volatile unsigned int *)GPIOC_ODR;

	/* Enable the clock for GPIOC */
	*pRccApb2Enr |= (1<<4);
	/* Set the GPIOC 13 port to be output mode */
    *pGpiocCrH |= (1<<20);
	
	while (1)
	{
		/* Only modify the required bit: |=, &= */
	    *pGpiocOdr |= (unsigned int)(1<<13);
		delay(100000);
	    *pGpiocOdr &= (unsigned int)~(1<<13);
		delay(100000);
	}
	return 0;
}
```
>start.s
```c
                PRESERVE8
                THUMB

; Vector Table Mapped to Address 0 at Reset
                AREA    RESET, DATA, READONLY
                EXPORT  __Vectors

__Vectors       DCD     0			               ; Top of Stack
                DCD     Reset_Handler              ; Reset Handler

                AREA    |.text|, CODE, READONLY
                
; Reset handler
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
                IMPORT  main
                LDR     SP, =(0x20000000 + 0x100)
                BL		main
                ENDP
					
				END
```
- after the device is powered and booted
	- `Reset_Handler` is the code to be executed
		- in which the function `main` is branched into
			- stack pointer is assigned with a valid address before branching
				- because the function execution requires the stack to run code in

## Memory Map
> ARM arch
- device is memory mapped
	- which means the device can be accessed by the address just like the normal memory
![Z - assets/images/Pasted image 20240228213033.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240228213033.png)

> X86 arch
- memory and I/O device are not mapped in the same space
	- memory
	- I/O device is mapped in a separate memory space
![Z - assets/images/Pasted image 20240228213153.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240228213153.png)
	-  obviously, two memory spaces are **overlapped** !

## RISC vs CISC
- ARM芯片采用精简指令集
	- 对于内存只有读和写指令
![Z - assets/images/Pasted image 20240228213844.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240228213844.png)
- X86芯片采用复杂指令集
![Z - assets/images/Pasted image 20240228213952.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240228213952.png)
- 复杂指令被封装成单个指令

## CPU寄存器
- CPU可以直接访问
	- 外围设备的寄存器访问需要通过地址
		- 将外部寄存器的地址赋给CPU内部寄存器r0
		- 使用r0所存储的地址来读取数据，并存入另一个内部寄存器
- 16个寄存器
	- 前13个为32-bit的通用寄存器
	- R13 - stack pointer
	- R14 - Link Register （saving the return address）
	- R15 - Program Counter
- xPSR - program status register
	- e.x. comparison result saved in the APSR (application program status register)

## ARM，Thumb，Thumb2 指令集
- ARM指令集：32-bit
	- 更高效，需要更多的空间
- Thumb指令集：16-bit
	- 节省空间
- 控制EPSR的T bit来在ARM和Thumb指令集间切换
- 多个程序使用不同的指令编写
	- PC程序计数器的bit0置1, CPU则进入Thumb状态，置0，CPU则进入ARM状态
- `CODE32/CODE16/THUMB`
```c
        AREA    ChangeState, CODE, READONLY
        CODE32
                             ; This section starts in ARM state
        LDR     r0,=start+1  ; Load the address and set the
                             ; least significant bit
        BX      r0           ; Branch and exchange instruction sets
                             
                             ; Not necessarily in same section
        CODE16               ; Following instructions are Thumb
start   MOV     r1,#10       ; Thumb instructions
```
- reference: https://developer.arm.com/documentation/dui0068/b/Directives-Reference/Miscellaneous-directives/CODE16-and-CODE32
- Thumb2指令集：支持ARM和Thumb指令集
	- 混合编程，不需要显示地进行指令集切换
		- 更高效 - CPU不需要频繁地进行状态切换
- Unified Assembly Language，UAL，统一汇编语言
	- 数据处理指令：`Operation{cond}{S} Rd, Rn, Operand2`
![Z - assets/images/Pasted image 20240229053249.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240229053249.png)
- 立即数
	- `Mov R1, #val`
		- 指令本身是16或者32位，因此`val`只能是特定数值，也就是立即数
			- 立即数定义![Z - assets/images/Pasted image 20240229053858.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240229053858.png)
			- reference: https://alisdair.mcdiarmid.org/arm-immediate-value-encoding/
	- ARM data processing instruction
	 ![Z - assets/images/Pasted image 20240229060011.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240229060011.png)
		- Operand2 is an immediate value, when the Bit25 is set to 1
		- 4-bit rotation (right circular rotate, why it is right?) and 8-bit number
			- 4-bit rotation is multiplied by 2 to enlarge the range
		 ![Z - assets/images/Pasted image 20240229060219.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240229060219.png)
		 - which values can be represented?
			 - `0x00 - 0xFF`
			 - rotated constant must be within `0x00 - 0xFF`
			 - even rotation
		- easy way to check whether the value is an immediate ?
			- `LDR R0, =VAL`

## 内存访问指令
> LDR / LDM / STR / STM
```c
		MOV		R0, #0x20000
		MOV		R1, #0x10
		MOV		R2, #0x12
		STR		R2, [R0]              ; R2的值存到R0所示地址
		STR		R2, [R0, #4]          ; R2的值存到R0+4所示地址
		STR		R2, [R0, #8]!         ; R2的值存到R0+8所示地址, R0=R0+8
		STR		R2, [R0, R1]          ; R2的值存到R0+R1所示地址
		STR		R2, [R0, R1, LSL #4]  ; R2的值存到R0+(R1<<4)所示地址
		STR		R2, [R0], #0X20       ; R2的值存到R0所示地址, R0=R0+0x20
		MOV		R2, #0x34
		STR		R2, [R0]              ; R2的值存到R0所示地址
		LDR		R3, [R0], +R1, LSL #1 ; R3的值等于R0+(R1<<1)所示地址上的值
```
- `LSL` - left shift instruction
```c
		MOV		R1, #1
		MOV		R2, #2
		MOV		R3, #3
		MOV		R0, #0x20000
		STMIA	R0,		{R1-R3} ; R1,R2,R3分别存入R0,R0+4,R0+8地址处
		ADD		R0, R0, #0x10
		STMIA	R0!, {R1-R3} ; R1,R2,R3分别存入R0,R0+4,R0+8地址处, R0=R0+3*4
```
- 低地址内存中存放低序号寄存器中的数值
	- STMDA (decrease after)![Z - assets/images/Pasted image 20240229213016.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240229213016.png)
```c
		MOV		R1, #1
		MOV		R2, #2
		MOV		R3, #3
		MOV		R0, #0x10000
		STR		R1, [R0]
		STR		R2, [R0, #4]
		STR		R3, [R0, #8]
		LDMIA	R0, {R1-R3}
```
- LDMIA![Z - assets/images/Pasted image 20240229213939.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240229213939.png)

## Stack 
- **4** 种模式的栈
	- full / empty
		- full: adjust the stack pointer and then push the data
		- empty: push the data and then adjust the stack pointer
	- ascending / descending
		- ascending: stack pointer increases
		- descending: stack pointer decreases
- full ascending / full descending / empty ascending / empty descending
- most widely used
	- full descending stack
```c
		MOV		R1, #1
		MOV		R2, #2
		MOV		R3, #3
		MOV		SP, #0x20000
		STMFD	SP!, {R1-R3}
```
- `FD` - full descending

## 数据处理指令
> ADD / SUB / AND / ORR / BIS ...
```c
		MOV		R2, #1
		MOV		R3, #2
		ADD		R1, R2, #15
		SUB		R4, R1, R2
		
		LDR		R0, =0xFFFFFFFF
		AND		R0, R0, #0x10
		LDR		R1, =0xFF00FF00
		LDR		R2, =0x00FF00FF
		ORR		R3, R1, R2
		
		LDR		R4, =0xFFFFFFFF
		bic		R4, R4, #0x10
		
		CMP		R1, R2
		MOVNE	R5, #0x1000
		
		LDR		R6, =0x10
		LDR		R7, =0x11
		TST		R6, R7
```

## 跳转指令
```c
		BL		Delay
		MOV		R1, #1
Delay
		MOV		R0, #5
Loop
		SUBS		R0, R0, #1
		BNE		Loop
		;		end of the loop
		MOV		PC, LR
```
- 直接使用pc
```c
		ADR		LR, Ret
		ADR		PC, Delay
Ret
		MOV		R1, #1
Delay
		MOV		R0, #5
Loop
		SUBS		R0, R0, #1
		BNE		Loop
		;		end of the loop
		MOV		PC, LR
```
> LR register needs to be configured in the beginning before branching into another function

## 伪指令转换
- 伪指令
```c
                LDR     SP, =(0x20000000 + 0x100)
```
- 转换后的真正指令
```c
    Reset_Handler
        0x080000b4:    f8dfd004    ....    LDR      sp,[pc,#4] ; [0x80000bc] = 0x20000100
        0x080000b8:    f000f860    ..`.    BL       main ; 0x800017c
    $d
        0x080000bc:    20000100    ...     DCD    536871168
```
	- `(0x20000000 + 0x100)`被存入在临时内存中
	- 伪指令；告诉编译器分配内存中的某个临时地址来存储某个数值？？？
		- 取决于该数值是否可以表示为立即数
			- but，SP好像是个exception，LDR指令永远都是暂存某个数值
- experiment
 > 立即数
```c
				LDR		R0, =0x1000
--------------------------------------------------------
        0x080000b4:    f44f5080    O..P    MOV      r0,#0x1000
```
> 非立即数
```c
				LDR		R0, =0x10011111
--------------------------------------------------------
        0x080000b4:    4802        .H      LDR      r0,[pc,#8] ; [0x80000c0] = 0x10011111
        ......
        0x080000c0:    10011111    ....    DCD    268505361
```
- summary
	- LDR伪指令
		- operand是立即数
			- 直接转换为MOV指令
		- operand不是立即数
			- 暂存在某一地址的内存中，并通过读取该内存取得数值
	- 上述过程取决于架构
		- 编译器会根据架构来进行机器码和汇编的对应转换

## 不同指令集的机器码
- Thumb-2 指令集
![Z - assets/images/Pasted image 20240302174228.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240302174228.png)
- 对应的机器码
```c
0x080000b4:    f8dfd004    ....    LDR      sp,[pc,#4] ; [0x80000bc] = 0x20000100
--------
1111,1000,1101,1111,1101,0000,0000,0100
<Rt> -> SP -> R13 -> 1101
imm12 -> #4 -> 0000,0000,0100
```

## ARM 流水线
- ARMV7 架构
![Z - assets/images/Pasted image 20240302174708.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240302174708.png)
> ARM指令集，PC 指向当前地址 + 8
- ARMV
![Z - assets/images/Pasted image 20240302174857.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240302174857.png)
> Thumb指令集，PC指向当前地址 + 4
![Z - assets/images/Pasted image 20240302175133.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240302175133.png)

## ARM-THUMB Procedure Call Standard
> define how parameters can be passed to a routine (function)
- `r0-r3`
	- caller and callee pass parameters in
- `r4-r11`
	- can be used in the function
	- needs to be pushed before the start of the function, and popped after the function finishes ?
![Z - assets/images/Pasted image 20240302210044.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240302210044.png)

## 阅读反汇编代码
- dis-assembly code
```c
    mymain
        0x0800002c:    b580        ..      PUSH     {r7,lr}
        0x0800002e:    b084        ..      SUB      sp,sp,#0x10
        0x08000030:    f2410018    A...    MOV      r0,#0x1018
        0x08000034:    f2c40002    ....    MOVT     r0,#0x4002
        0x08000038:    9003        ..      STR      r0,[sp,#0xc]
        0x0800003a:    f2410004    A...    MOV      r0,#0x1004
        0x0800003e:    f2c40001    ....    MOVT     r0,#0x4001
        0x08000042:    9002        ..      STR      r0,[sp,#8]
        0x08000044:    f241000c    A...    MOV      r0,#0x100c
        0x08000048:    f2c40001    ....    MOVT     r0,#0x4001
        0x0800004c:    9001        ..      STR      r0,[sp,#4]
        0x0800004e:    9903        ..      LDR      r1,[sp,#0xc]
        0x08000050:    6808        .h      LDR      r0,[r1,#0]
        0x08000052:    f0400010    @...    ORR      r0,r0,#0x10
        0x08000056:    6008        .`      STR      r0,[r1,#0]
        0x08000058:    9902        ..      LDR      r1,[sp,#8]
        0x0800005a:    6808        .h      LDR      r0,[r1,#0]
        0x0800005c:    f4401080    @...    ORR      r0,r0,#0x100000
        0x08000060:    6008        .`      STR      r0,[r1,#0]
        0x08000062:    e7ff        ..      B        0x8000064 ; mymain + 56
        0x08000064:    9901        ..      LDR      r1,[sp,#4]
        0x08000066:    6808        .h      LDR      r0,[r1,#0]
        0x08000068:    f4405000    @..P    ORR      r0,r0,#0x2000
        0x0800006c:    6008        .`      STR      r0,[r1,#0]
        0x0800006e:    f24860a0    H..`    MOV      r0,#0x86a0
        0x08000072:    f2c00001    ....    MOVT     r0,#1
        0x08000076:    9000        ..      STR      r0,[sp,#0]
        0x08000078:    f7ffffcc    ....    BL       delay ; 0x8000014
        0x0800007c:    9800        ..      LDR      r0,[sp,#0]
        0x0800007e:    9a01        ..      LDR      r2,[sp,#4]
        0x08000080:    6811        .h      LDR      r1,[r2,#0]
        0x08000082:    f4215100    !..Q    BIC      r1,r1,#0x2000
        0x08000086:    6011        .`      STR      r1,[r2,#0]
        0x08000088:    f7ffffc4    ....    BL       delay ; 0x8000014
        0x0800008c:    e7ea        ..      B        0x8000064 ; mymain + 56
        0x0800008e:    0000        ..      MOVS     r0,r0
```
- C source code
```c
int mymain(void)
{
	volatile unsigned int *pRccApb2Enr;
	volatile unsigned int *pGpiocCrH;
	volatile unsigned int *pGpiocOdr;
	
	pRccApb2Enr = (volatile unsigned int *)RCC_APB2ENR;
	pGpiocCrH   = (volatile unsigned int *)GPIOC_CRH;
	pGpiocOdr   = (volatile unsigned int *)GPIOC_ODR;
	
	*pRccApb2Enr |= (1<<4);
	*pGpiocCrH |= (1<<20);
	while (1)
	{
	    *pGpiocOdr |= (unsigned int)(1<<13);
		delay(100000);
	    *pGpiocOdr &= (unsigned int)~(1<<13);
		delay(100000);
	}
	return 0;
}
```

```c
        0x0800002c:    b580        ..      PUSH     {r7,lr}
        ; 0x20000FFC LR
        ; 0x20000FF8 r7
        
        0x0800002e:    b084        ..      SUB      sp,sp,#0x10
        ; 3 local unsigned int variable，reserve space for them
        ; sp = 0x20000FE8

        0x08000030:    f2410018    A...    MOV      r0,#0x1018
        0x08000034:    f2c40002    ....    MOVT     r0,#0x4002
        0x08000038:    9003        ..      STR      r0,[sp,#0xc]
        ; save 0x40011018 to 0x20000FF4
        
		0x0800003a:    f2410004    A...    MOV      r0,#0x1004
        0x0800003e:    f2c40001    ....    MOVT     r0,#0x4001
        0x08000042:    9002        ..      STR      r0,[sp,#8]
        ; save 0x40011004 to 0x20000FF0

        0x08000044:    f241000c    A...    MOV      r0,#0x100c
        0x08000048:    f2c40001    ....    MOVT     r0,#0x4001
        0x0800004c:    9001        ..      STR      r0,[sp,#4]
        ; save 0x4001100C to 0x20000FEC

	    0x0800004e:    9903        ..      LDR      r1,[sp,#0xc]
        0x08000050:    6808        .h      LDR      r0,[r1,#0]
        0x08000052:    f0400010    @...    ORR      r0,r0,#0x10
		0x08000056:    6008        .`      STR      r0,[r1,#0]
        ; read the value from the register at the address of 0x40011018
        ; why first loading to r1 and then r0? instead of directly loading it to r0?
        ; because the register address is required when loading and saving, r0 is changed
        ; logic or with 0x10 and save the result to the register

        0x08000058:    9902        ..      LDR      r1,[sp,#8]
        0x0800005a:    6808        .h      LDR      r0,[r1,#0]
        0x0800005c:    f4401080    @...    ORR      r0,r0,#0x100000
        0x08000060:    6008        .`      STR      r0,[r1,#0]
        ; similar to the code above, but manipulate the register at the address of 0x40011004
        ; ...

        0x08000062:    e7ff        ..      B        0x8000064 ; mymain + 56
        ; while loop - first line of code ?

        0x08000064:    9901        ..      LDR      r1,[sp,#4]
        0x08000066:    6808        .h      LDR      r0,[r1,#0]
        0x08000068:    f4405000    @..P    ORR      r0,r0,#0x2000
        0x0800006c:    6008        .`      STR      r0,[r1,#0]
		; similar to the code above, but manipulate the register at the address of 0x4001100C
        ; ...

        0x0800006e:    f24860a0    H..`    MOV      r0,#0x86a0
        0x08000072:    f2c00001    ....    MOVT     r0,#1
        0x08000076:    9000        ..      STR      r0,[sp,#0]
        0x08000078:    f7ffffcc    ....    BL       delay ; 0x8000014
        ; pass the parameters 0x18610
        ; branch into the delay function
        ; STR r0, [sp, #0] - optimization ? reuse the 100000 number ?

        0x0800007c:    9800        ..      LDR      r0,[sp,#0]
        ; restore the value 100000 temporaily saved in the r0 register which
        ; will be used when calling the delay again ?
        ; wait, this is actually the parameter (first word) saved for the stack of delay function ?
    
        0x0800007e:    9a01        ..      LDR      r2,[sp,#4]
        0x08000080:    6811        .h      LDR      r1,[r2,#0]
        0x08000082:    f4215100    !..Q    BIC      r1,r1,#0x2000
        0x08000086:    6011        .`      STR      r1,[r2,#0]
        ; clear the 13th bit
        ; NOTE: compiler optimizes the AND with BIC (simpler)

		0x08000088:    f7ffffc4    ....    BL       delay ; 0x8000014
		; branch into the delay, with the parameters restored in r0 before

		0x0800008c:    e7ea        ..      B        0x8000064 ; mymain + 56
		; keep looping in while
```

```c
    __Vectors
        0x08000000:    00000000    ....    DCD    0
        ; top of stack, SP
        0x08000004:    08000009    ....    DCD    134217737
        ; first instruction address pointed by PC
        ; 0x80000009 = 0x80000008 + 0x1
        ; last bit is set to 1 to indicate the Thumb instruction sets are used
```

## 汇编代码blink led
```c
                PRESERVE8
                THUMB

; Vector Table Mapped to Address 0 at Reset
                AREA    RESET, DATA, READONLY
                EXPORT  __Vectors

__Vectors       DCD     0			               ; Top of Stack
                DCD     Reset_Handler              ; Reset Handler


                AREA    |.text|, CODE, READONLY
                
; Reset handler
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
				
				; Enable GPIO clock
				LDR		R0, =0x40021018
				LDR		R1, [R0]
				ORR		R1, R1, #0x10
				STR		R1, [R0]
				
				; Configure control register
				LDR		R0, =0x40011004
				LDR		R1, [R0]
				ORR		R1, R1, #0x100000
				STR		R1, [R0]
Loop
				; Set output to high
				LDR		R0, =0x4001100C
				LDR		R1, [R0]
				ORR		R1, R1, #0x2000
				STR		R1, [R0]
				
				; Delay
				LDR		R0, =100000
				BL		Delay
				
				; Set output to low
				LDR		R0, =0x4001100C
				LDR		R1, [R0]
				BIC		R1, R1, #0x2000
				STR		R1, [R0]
				
				; Delay
				LDR		R0, =100000
				BL		Delay
				
				; Looping ...
				B Loop
				ENDP

Delay
				; decrease R0
				; compare R0 with 0
				; loop until R0 = 0
				SUBS	R0, R0, #1
				BNE		Delay
				MOV		PC,	LR
				
				END
```

## LED controlled by the key
```c
/*
 * Address of the PB14 and PC13
 */
#define RCC_APB2ENR (0x40021000 + 0x18)

#define GPIOC_BASE  (0x40011000)
#define GPIOC_CRH   (GPIOC_BASE + 0x04)
#define GPIOC_ODR   (GPIOC_BASE + 0X0C)

#define GPIOB_BASE  (0x40010C00)
#define GPIOB_CRH   (GPIOB_BASE + 0x04)
#define GPIOB_IDR   (GPIOB_BASE + 0x08)

/** Main Entry
 * @brief  
 * @note   
 * @param  argc: 
 * @param  argv: 
 * @retval 
 */
int main(void)
{
    volatile unsigned int *pRccApb2En;
    volatile unsigned int *pGpioCCtrH;
    volatile unsigned int *pGpioCOdr;
    
    volatile unsigned int *pGpioBCtrH;
    volatile unsigned int *pGpioBIdr;

    pRccApb2En = (volatile unsigned int *)RCC_APB2ENR;
    pGpioCCtrH = (volatile unsigned int *)GPIOC_CRH;
    pGpioCOdr = (volatile unsigned int *)GPIOC_ODR;

    pGpioBCtrH = (volatile unsigned int *)GPIOB_CRH;
    pGpioBIdr = (volatile unsigned int *)GPIOB_IDR;

    /* Enable the clock for IOPB and IOPC */
    *pRccApb2En |= (1<<3) | (1<<4);

    /* Configure PC13 to be output */
    *pGpioCCtrH |= (1<<20);

    /* Configure PB14 to be floating input */
    *pGpioBCtrH &= (0x00<<24);
    *pGpioBCtrH |= (0x01<<26);

    /* main loop */
    while (1)
    {
        /* Read the key input register */
        int keyEn = (*pGpioBIdr) & (1<<14);
        if (keyEn == 0)
        {
			/* Set the PC13 output to be low */
            *pGpioCOdr &= (unsigned int)~(1<<13);
        }
        else
        {
            /* Set the PC13 output to be high */
            *pGpioCOdr |= (unsigned int)(1<<13);
        }
    }

    return 0;
}
```
> assembly
```c
                PRESERVE8
                THUMB


; Vector Table Mapped to Address 0 at Reset
                AREA    RESET, DATA, READONLY
                EXPORT  __Vectors

__Vectors       DCD     0			               ; Top of Stack
                DCD     Reset_Handler              ; Reset Handler


                AREA    |.text|, CODE, READONLY
                
; Reset handler
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]
				
				; Enable the clock for IOPB and IOPC
				LDR		R0, =0x40021018
				LDR		R1, [R0]
				ORR		R1, R1, #(1<<3)
				ORR		R1, R1, #(1<<4)
				STR		R1, [R0]
				
				; Configure PC13 to be output
				LDR		R0, =0x40011004
				LDR		R1, [R0]
				ORR		R1, R1, #0x100000
				STR		R1, [R0]
				
				; Configure PB14 to be input
				LDR		R0, =0x40010C04
				LDR		R1, [R0]
				AND		R1, R1, #(0x00<<24)
				ORR		R1, R1, #(0x01<<26)
				STR		R1, [R0]

Loop
				; Read the PB14 input register
				LDR		R0, =0x40010C08
				LDR		R1, [R0]
				AND		R1, R1, #(1<<14)
				CBNZ	R1, LED_OFF
				;B		LED_ON
				; Set PC13 output to be low
				LDR		R0, =0x4001100C
				LDR		R1, [R0]
				BIC		R1, R1, #0x2000
				STR		R1, [R0]
				
				; Continue looping
				B 		Loop

LED_OFF
				; Set PC13 output to be high
				LDR		R0, =0x4001100C
				LDR		R1, [R0]
				ORR		R1, R1, #0x2000
				STR		R1, [R0]
				
				; Continue looping
				B		Loop

                ENDP

				END
```
- `CBNZ`
	- compare and branch on non-zero
	- reference: [ARM Compiler toolchain Assembler Reference Version 5.03](https://developer.arm.com/documentation/dui0489/i/arm-and-thumb-instructions/cbz-and-cbnz)

## UART
- `Universal Asynchronous Receiver and Transmitter`
- three-wire connection
	- TXD
	- RXD
	- GND
- typical connection
![Z - assets/images/Pasted image 20240303150849.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240303150849.png)
- configuration parameters
	- baud rate
	- packet frame
		- data bits
		- stop bits
		- parity bit
		- flow control 
- transmission process
![Z - assets/images/Pasted image 20240303151708.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240303151708.png)
- ARM: start bit is the logic `0`
- client在数据位的中间读取引脚数据
	- 开始传输数据的时刻
	- 每一位数据的传输时间 （baud rate约束）
	- 当前数据的位数
- 校验位
	- 数据位 + 校验位
	- 奇偶校验
	- why：早期电子技术不够完善，传输信号质量
		- 现代嵌入式系统中，电子传输信号质量较高，这一校验位可以省去
- 停止位
	- 高电平（相对起始位是低电平）
	- 持续时间可配置，例如1位时间
- 逻辑电平
![Z - assets/images/Pasted image 20240303152529.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240303152529.png)
- `RS-232`: 适合远距离传输，PC机中广泛使用
- `TTL`: ARM设备中使用
- `TTL/CMOS` -> `RS-232`: 电平转换芯片，通常是集成在嵌入式开发板上
![Z - assets/images/Pasted image 20240303161056.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240303161056.png)
- experiment
> uart.h
```c
#ifndef UART_H_
#define UART_H_

/* ======================= typedefs ========================== */
typedef unsigned int uint32_t ;

/* Easy manipulation of USART registers */
typedef struct
{
    volatile uint32_t SR;    /*!< USART Status register, Address offset: 0x00 */
    volatile uint32_t DR;    /*!< USART Data register,   Address offset: 0x04 */
    volatile uint32_t BRR;   /*!< USART Baud rate register, Address offset: 0x08 */
    volatile uint32_t CR1;   /*!< USART Control register 1, Address offset: 0x0C */
    volatile uint32_t CR2;   /*!< USART Control register 2, Address offset: 0x10 */
    volatile uint32_t CR3;   /*!< USART Control register 3, Address offset: 0x14 */
    volatile uint32_t GTPR;  /*!< USART Guard time and prescaler register, Address offset: 0x18 */
} USART;

/* ======================= prototypes ========================== */
void uart_init(void);
int uart_send(char c);
int uart_get(void);

#endif

```
> uart.c
```c
/*====================== includes ========================== */
#include "uart.h"

/* ======================= defines ========================== */
#define USART1_BASE_ADDR 0x40013800
#define RCC_APB2ENR	(0x40021000 + 0x18)
#define GPIOA_BASE (0x40010800)
#define GPIOA_CRH (GPIOA_BASE + 0x04)
#define _IO (volatile uint32_t *)
#define USART1_DIV_MANTISSA 4U
#define USART1_DIV_FRACTION 5U

/* ======================= functions ========================== */

/**!SECTION: UART Initialization
 * @brief  Initialize UART
 */
void uart_init(void)
{
    /* USART1 */
    volatile USART *pUSART1 = (volatile USART *)USART1_BASE_ADDR;

    volatile uint32_t *pRCCAPB2ENR =  (volatile uint32_t *) RCC_APB2ENR;
    volatile uint32_t *pGPIOACRH =  (volatile uint32_t *) GPIOA_CRH;
    
    /* Enable clock for USART1  */
    *pRCCAPB2ENR |= (1U << 2) | (1U << 14);

    /* Configure PA9  to output mode USART1 */
    *pGPIOACRH &= ~((3U << 4) | (3U << 6)); /* Reset value is 0x4444,4444, clear bits to ensure no conflicts */
    *pGPIOACRH |= (1U << 4) | (2U << 6); /* MODE(0x01): output mode, max speed 10 MHz, CNF(0x10): push-pull ? */

    /* Configure PA10 to input mode USART1 */
    *pGPIOACRH &= ~((3U << 8) | (3U << 10)); /* Reset value is 0x4444,4444, clear bits to ensure no conflicts */
    *pGPIOACRH |= (0U << 8) | (1U << 10); /* MODE(0x00): input mode, CNF(0x01): floating input ? */

    /* Set baud rate 
     * 115200 = 8M (fclk) / 16 / USARTDIV
     * USARTDIV = 4.34
     * DIV_Mantissa = 4
     * DIV_Fraction = 0.34 * 16 = 5.44 = 5
     * Actual baud rate = 115942
     */
    pUSART1->BRR = (USART1_DIV_MANTISSA << 4) | (USART1_DIV_FRACTION);

    /* Set data frame and enable the USART1 */
    pUSART1->CR1 = (1U << 13) | (0U << 12) | (0U << 10) | (1U << 3) | (1U << 2);
    pUSART1->CR2 &= ~(3U << 12);
}

/**!SECTION UART send 
 * 
*/
int uart_send(char c)
{
    /* USART1 */
    volatile USART *pUSART1 = (volatile USART *)USART1_BASE_ADDR;

    /* Wait until the data in the TX buffer have been sent to the shift register */
    while  ((pUSART1->SR & (1U << 7)) == 0);

    /* Put data into the TX buffer */
    pUSART1->DR = c;

    return c;
}

/**!SECTION UART receive
 * 
 */
int uart_get(void)
{
    /* USART1 */
    volatile USART *pUSART1 = (volatile USART *)USART1_BASE_ADDR;

    /* Wait until the data in the RX buffer are ready to read */
    while ((pUSART1->SR & (1U << 5)) == 0);

    return pUSART1->DR & 0xFF;
}
```
> main.c
```c
/*====================== includes ========================== */
#include "uart.h"

/* ======================= defines ========================== */
#define SIZE_OF_ARRAY(x) (sizeof(x) / sizeof(x[0]))
#define size_t unsigned int

/* ======================= globals ========================== */
/* Can't use global variable ? */
// static USART *pUSART1 = (USART *)USART1_BASE_ADDR;

int main(void)
{
    const char pData[] = "Sending from Maxin's powerful desktop!\n\r";
    unsigned int lenOfData = SIZE_OF_ARRAY(pData);

    /* Initialize UART */
    uart_init();

    /* Send all characters one by one */
    for (size_t i = 0; i < lenOfData; ++i)
    {
        uart_send(pData[i]);
    }

    return 0;
}
```

## Find the stdlib used in GCC
```bash
echo 'main(){}'| gcc -E -v -
```

## Code relocate
> experiment
```c
/*====================== includes ========================== */
#include "util.h"
#include "uart.h"
#include "string.h"

/* ======================= globals ========================== */
/* Can't use global variable ? */
// static USART *pUSART1 = (USART *)USART1_BASE_ADDR;
char g_Char = 'A';
const char g_CharConst = 'B';
const char RECEIVED_MSG[] = "Received => ";

/* ======================= defines ========================== */
#define SIZE_OF_ARRAY(x) (sizeof(x) / sizeof(x[0]))
#define size_t unsigned int
	
static inline void uart_receive(void)
{
		uart_print(RECEIVED_MSG, SIZE_OF_ARRAY(RECEIVED_MSG));
}

static inline void uart_println(const char c)
{
		/* Send the given char */
		uart_send(c);
		/* Move the cursor to the start of the next line */
		uart_send('\n');
		uart_send('\r');
}

int main(void)
{
    const char pData[] = "Sending from Maxin's powerful desktop!\n\r";
    unsigned int lenOfData = SIZE_OF_ARRAY(pData);
		char c;

    /* Initialize UART */
    uart_init();
	
		/* Send two global char */
		uart_println(g_Char);
		uart_println(g_CharConst);
	
		/* Print the addresses of two global objects */
		put_s_hex("Address of g_Char = ", (void*)&g_Char - (void*)0);
		put_s_hex("Address of g_CharConst = ", (void*)&g_CharConst - (void*)0);
	

#if 0
    /* Send all characters one by one */
    for (size_t i = 0; i < lenOfData; ++i)
    {
        uart_send(pData[i]);
    }
#endif
		
		while (1)
		{
				/* Fetch received characters */
				c = uart_get();
				/* Print "Received => " to improve readability */
			  uart_receive();
				uart_println(c);
		}

    return 0;
}

```
- `uart_println(g_Char);` doesn't print the character of `A`
- address of global objects (const VS non-const)![Z - assets/images/Pasted image 20240303211515.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240303211515.png)
	- `g_Char`: saved in `RAM`
		- uninitialized, no default value
			- **that's why A doesn't appear!!!**
	- `g_CharConst`": saved in `ROM`
		- initialized, default value as assigned
- different code section ![Z - assets/images/Pasted image 20240303211628.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240303211628.png)
	- `rodata` VS `data`
- `start.s` needs to do more
	- 重定位
		- 代码重定位
			- 代码烧录到ROM中
		- 数据重定位
			- 将ROM程序中可读可写的全局变量复制到RAM中（包含初始值）
![Z - assets/images/Pasted image 20240303212538.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240303212538.png)
- scatter file
```c
LR_IROM1 0x08000000 0x00010000  {    ; load region size_region
  ER_IROM1 0x08000000 0x00010000  {  ; load address = execution address
   *.o (RESET, +First)
   *(InRoot$$Sections)
   .ANY (+RO)
   .ANY (+XO)
  }
  RW_IRAM1 0x20000000 0x00005000  {  ; RW data
   .ANY (+RW +ZI)
  }
}
```

- load address
	- execution address
		- relocate if load address != execution address
- `*.o (RESET, +First)` 
	- all `RESET` sections contained in object files (*.o)
		- to be placed at the beginning of this region (`ER_IROM1`)
		- ONLY found this in `start.o`
```c
                AREA    RESET, DATA, READONLY
                EXPORT  __Vectors

__Vectors       DCD     0			               ; Top of Stack
                DCD     Reset_Handler              ; Reset Handler

  RW_IRAM1 0x20000000 0x00005000  {  ; RW data
   .ANY (+RW +ZI)
  }
}
```
- 保存在RAM中的全局变量未进行初始化的原因
	- reference: [[012] [STM32] 代码重定位与清除BSS段深入分析_stm32 0x20001000-CSDN博客](https://blog.csdn.net/kouxi1/article/details/123492797)
```markdown
MDK环境下，STM32启动文件中的__main帮我们做了重定位等一系列操作，如果不使用编译器提供的__main函数（主函数名称为main也会默认调用__main），则**RW-DATA中变量的值仅保存在ROM中**，没有将值复制到RAM中，但程序访问变量时，是去从该变量所在RAM内存地址处加载值的，但此地址却并未初始化，所以读取的数值为junk。
```

> .bss清零
- 全局变量
	- 无initializer
	- 数值为0的initializer
- 取决于变量占用的空间 - Keil环境下的优化
	- 超过8Bytes，编译器会将其放置在.bss
	- 不然，编译器会将其放置在.data
![Z - assets/images/Pasted image 20240305213406.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240305213406.png)
![Z - assets/images/Pasted image 20240305213504.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240305213504.png)
- 实际观察到，即使变量占据空间小于8Bytes，编译器还是将其放置在.bss段？
- **BSS段并不会放入bin文件中，否则也太浪费空间了，在使用BSS段里的变量之前，把BSS段所占据的内存清零就可以了。**
	- 执行地址，位于内存中（RAM）

- 使用绝对跳转指令（跳转地址为确定的链接地址）
```c
                ;BL		mymain
                LDR		R0, =mymain
				BLX		R0
```
> 反汇编
```c
        0x20000028:    4809        .H      LDR      r0,[pc,#36] ; [0x20000050] = 0x20000145
        0x2000002a:    4780        .G      BLX      r0
```
- 而不是相对跳转地址
	- 编译器翻译`BL    mymain` 为 `BL    [PC, #offset]`
		- 其中offset为链接时确定的地址偏移量
> 反汇编
```c
        0x20000028:    f000f88a    ....    BL       mymain ; 0x20000140
```

- pending question
	- start.s中的代码是在ROM中运行的？
- observation
	- ~~使用相对和绝对地址跳转命令得到的反汇编代码似乎都表明，程序最终跳转到RAM中运行
	- comment out进行code relocation的代码，分别使用相对跳转和绝对跳转，
		- 相对跳转正常工作，因为此时ROM中的代码在运行，绝对跳转不工作，因为RAM中对应地址处无代码
		- **因此证明了在进行code relocation以后，使用绝对跳转，程序正常运行，表明RAM中的代码在运行**
	- 反推在反汇编中看到的跳转地址
		- 反汇编中左侧的地址仅是链接地址，并非一定是程序最终运行时所处的地址
		- 链接地址取决于编译器（链接过程）
> start.s
```c
                PRESERVE8
                THUMB

; Vector Table Mapped to Address 0 at Reset
                AREA    RESET, DATA, READONLY
                EXPORT  __Vectors

__Vectors       DCD     0			               ; Top of Stack
                DCD     0x08000009 ;Reset_Handler              ; Reset Handler

                AREA    |.text|, CODE, READONLY
			
; Reset handler
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]				
				IMPORT  mymain
				IMPORT  relocate_memcpy
				IMPORT	relocate_memset
					
				IMPORT	|Image$$RW_IRAM1$$Base|
				IMPORT	|Image$$RW_IRAM1$$Length|
				IMPORT	|Load$$RW_IRAM1$$Base|
				
				IMPORT  |Image$$RW_IRAM1$$ZI$$Base|
				IMPORT  |Image$$RW_IRAM1$$ZI$$Length|

				IMPORT |Image$$ER_IROM1$$Base|
				IMPORT |Image$$ER_IROM1$$Length|
				IMPORT |Load$$ER_IROM1$$Base|

				; Set stack top address
                LDR     SP, =(0x20000000 + 0x1000)

				; Relocate text section (RW)
				LDR 	R0, = |Image$$ER_IROM1$$Base|    ; DEST
				LDR 	R1, = |Load$$ER_IROM1$$Base|     ; SORUCE
				LDR 	R2, = |Image$$ER_IROM1$$Length|  ; LENGTH
				BL		relocate_memcpy

				; Relocate data section (RW)
				LDR		R0,	= |Image$$RW_IRAM1$$Base| 		;Destination
				LDR		R1, = |Load$$RW_IRAM1$$Base|		;Source
				LDR		R2, = |Image$$RW_IRAM1$$Length|  	;Length
				BL		relocate_memcpy
				
				; Clear .bss
				LDR 	R0, = |Image$$RW_IRAM1$$ZI$$Base|       ; DEST
				LDR 	R1, = |Image$$RW_IRAM1$$ZI$$Length|     ; Length
				BL 		relocate_memset
								
                ;BL		mymain
                LDR		R0, =mymain
				BLX		R0
				
				ENDP
				END

```
- **NOTE**
	- `DCD     0x08000009`
		- **即使要进行code relocation，还是需要将PC初始指向ROM中，因为relocation的代码，亦即relocate_memcpy保存在ROM中**
> uart.scat
```c
; *************************************************************
; *** Scatter-Loading Description File generated by uVision ***
; *************************************************************

LR_IROM1 0x08000000 0x00010000  {    ; load region size_region
  ER_IROM1 0x20000000 {  ; load address != execution address
   *.o (RESET, +First)
   ;*(InRoot$$Sections)
   .ANY (+RO)
   .ANY (+XO)
  }
  RW_IRAM1 +0 {  ; RW data
   .ANY (+RW +ZI)
  }
}
```
> C函数实现数据和代码的重定位
```c
/**!SECTION
 * @fn     systemInit
 */
void systemInit(void)
{
    extern char Image$$RW_IRAM1$$Base[];
    extern int Image$$RW_IRAM1$$Length;
    extern char Load$$RW_IRAM1$$Base[];

    extern char Image$$RW_IRAM1$$ZI$$Base[];
    extern int Image$$RW_IRAM1$$ZI$$Length;

    extern char Image$$ER_IROM1$$Base[];
    extern int Image$$ER_IROM1$$Length;
    extern char Load$$ER_IROM1$$Base[];

    /* Relocate text section (RO) */
    memcpy(Image$$ER_IROM1$$Base, Load$$ER_IROM1$$Base, (size_t)&Image$$ER_IROM1$$Length);

    /* Clear .bss */
    memset(Image$$RW_IRAM1$$ZI$$Base, 0x00, (size_t)&Image$$RW_IRAM1$$ZI$$Length);

    /* Relocate data section (RW) */
    memcpy(Image$$RW_IRAM1$$Base, Load$$RW_IRAM1$$Base, (size_t)&Image$$RW_IRAM1$$Length);
}
```
- 参考: `section: 6.3.7 Methods of importing linker-defined symbols in C and C++` in `DUI0803J_armlink_user_guide.pdf`

## 中断与异常
![Z - assets/images/Pasted image 20240306065252.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240306065252.png)
- 中断也是异常的
	- CPU不是运行在正常执行指令的状态下
```markdown
* 初始化
  * 设置中断源，让它可以产生中断
  * 设置中断控制器(可以屏蔽某个中断，优先级)
  * 设置CPU总开关，使能中断

* 执行其他程序：正常程序
* 产生中断，举例：按下按键--->中断控制器--->CPU
* cpu每执行完一条指令都会检查有无中断/异常产生
* 发现有中断/异常产生，开始处理：
  * 保存现场
  * 分辨异常/中断，调用对于异常/中断的处理函数
  * 恢复现场
```
- `cpu每执行完一条指令都会检查有无中断/异常产生`
	- 中断控制器？
- 保存和恢复现场, 分辨中断源
	- 硬件实现，e.x. M3 / M4
		- 向量表中保存的是中断函数的地址
		- 主要是定义中断函数
	- 软件实现, e.x. A7
		- 向量表中保存的是跳转指令
		- CPU进入相应的异常模式，运行对应的跳转指令（如何选择？）
			- 软件实现对于中断源的分辨，现场的保存以及恢复

## 硬件保存现场
- 调用者保存的寄存器
	- `R0 - R3, R12, LR, PSR`
- 被调用者保存的寄存器
	- `R4 - R11`
- 保存现场时需要保存的寄存器
	- 理解：调用中断函数前进行保存，也就是调用者需要保存
	- `R0 - R3, R12, LR, PSR`
	- `PC`
		- 流水线特性，也就是下一条指令地址，返回地址
- `M3 / M4` 在调用异常处理函数前，会把`LR`设置为特殊值，称为`EXC_RETURN`
	- **`0xF0000000 ~ 0xFFFFFFFF`在ARM中被定义为不可执行地址区域**
- 硬件异常返回机制
	- `PC`寄存器中值为`EXC_RETURN`，恢复保存的寄存器值
- `EXC_RETURN` bit 2定义了哪个栈用来保存现场
	- `M3 / M4`有两个栈
		- 主栈
		- 线程栈
- Cortex A7
	- 不同模式下都有各自的`banked`寄存器
		- 节省了保存相应寄存器的时间
		- 都有属于自己的栈
			- 保存现场在对应模式的自己的栈中

## Experiment: UsageFault
- `Configurable fault status register (SCB_CFSR)`
	- `Bit 16 UNDEFINSTR: Undefined instruction usage fault:` 
		- **When this bit is set to 1, the PC value stacked for the exception return points to the undefined instruction.**
			- that's to say, this bit needs to be cleared, otherwise, the exception keeps happening because the LR always points to the undefined instruction ?
		- **The UFSR bits are sticky. This means as one or more fault occurs, the associated bits are set to 1. A bit that is set to 1 is cleared to 0 only by writing 1 to that bit, or by a reset.**
			- this explains why `pSCB->CFSR |= pSCB->CFSR;` does the job!
![Z - assets/images/Pasted image 20240306220343.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240306220343.png)
```c
                PRESERVE8
                THUMB

; Vector Table Mapped to Address 0 at Reset
                AREA    RESET, DATA, READONLY
                EXPORT  __Vectors
				IMPORT	HardFault_Handler
				IMPORT  UsageFault_Handler

__Vectors       DCD     0			               ; Top of Stack
                DCD     0x08000009 ;Reset_Handler              ; Reset Handler
				DCD     0                ; NMI Handler
                DCD     HardFault_Handler          ; Hard Fault Handler
                DCD     0          ; MPU Fault Handler
                DCD     0           ; Bus Fault Handler
                DCD     UsageFault_Handler_ASM         ; Usage Fault Handler
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                          ; Reserved
                DCD     0                ; SVCall Handler
                DCD     0           ; Debug Monitor Handler
                DCD     0                          ; Reserved
                DCD     0             ; PendSV Handler
                DCD     0            ; SysTick Handler
				

                AREA    |.text|, CODE, READONLY

; Reset handler
Reset_Handler   PROC
                EXPORT  Reset_Handler             [WEAK]				
				IMPORT  mymain
				IMPORT  systemInit
				IMPORT	uart_init
				IMPORT	UsageFaultEnable
				
				; Set stack top address
                LDR     SP, =(0x20000000 + 0x1000)
				
				BL		uart_init
				BL		UsageFaultEnable
				
				; Experiment to see certain registers are saved
				LDR		R0, =0x00000001
				LDR		R1, =0x00000010
				LDR		R2, =0x00000100
				LDR		R3, =0x00001000
				LDR		R12, =0x00010000
				LDR		LR, =0x00100000 

				DCD		0xFFFFFFFF
								
                ;BL		mymain
                LDR		R0, =mymain
				BLX		R0
				
				ENDP

; UsageFault_Handler with stack as the argument
UsageFault_Handler_ASM PROC
				; To not override EXC_RETURN value saved in the LR register when jumping into the exception handler
				MOV	R0,	SP
				B UsageFault_Handler
				ENDP

				END
```
- **NOTE**: `B UsageFault_Handler` is used to not overwrite the special value of `EXC_RETURN` saved in the `LR` register
	- difference between `B` and `BL`

## Experiment SVCallFault
- different from the usagefault
	- PC doesn't stuck at the undefined instruction which causes the exception
		- but instead points to the next instruction
![Z - assets/images/Pasted image 20240307231418.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240307231418.png)
- associated map file
![Z - assets/images/Pasted image 20240307231513.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240307231513.png)
- Linux系统中使用SVC指令，故意触发异常处理函数，从而进入内核态来调用各种系统服务