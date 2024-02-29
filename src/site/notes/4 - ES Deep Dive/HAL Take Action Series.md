---
{"dg-publish":true,"permalink":"/4-es-deep-dive/hal-take-action-series/","noteIcon":"","created":"2024-02-26T20:34:12.366+01:00","updated":"2024-02-29T05:36:42.498+01:00"}
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