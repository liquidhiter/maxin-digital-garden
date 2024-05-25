---
{"dg-publish":true,"permalink":"/03-es-deep-dive/2-free-rtos/","noteIcon":"","created":"2024-03-09T22:13:22.289+01:00","updated":"2024-05-25T20:16:13.464+02:00"}
---

## Introduction
- 裸机开发的缺陷
	- 单个程序执行时间过长导致其它程序不能及时响应
		- 解决方案
			- 前后台
				- 本质上还是中断驱动
			- 事件驱动
				- 依然避免不了上述问题：另一个事件依然可能得不到响应
			- 中断驱动
				- 问题依旧
	- 可行的解决方案
		- 尽可能减少单个程序的执行时间
			- 状态驱动：将某个程序设计成一个状态机，某一段运行时只执行对应状态下的功能
			- 缺点：
				- 将程序拆分成多个状态有时很困难
				- 程序设计更复杂

## RTOS
- 多个任务之间的依赖关系
	- 视频中举例，两个任务A和B，B任务的执行取决于A任务的计算结果
	- A任务的计算时间由于系统的定时切换double了
	- 解决方案：
		- B任务在计算结果未得出时进入休眠
		- A任务在得出计算结果后唤醒任务B
- 任务有优先级
	- 优先级高的任务必须先执行
		- 被打断的任务是在执行完当前指令？
- 同步与互斥
	- I.互斥
```c
void uart_print(char *str)
{
    g_canuse--;            ① 减一
    if( g_canuse == 0 )    ② 判断
    {
        printf(str);     ③ 打印
    }
    g_canuse++;          ④ 加一
}
```
- task A execution:
	- `g_canuse--`
		- read the value from the register
		- decrement by one
		- write back the value
	- switching to task B can happen before write back
		- in this case, the `g_canuse` is still equal to 1
		- task B can print something ...
	- summarization
		- `基于多任务系统编写程序时，访问公用的资源的时候要考虑“互斥操作”。`
	- II. 同步
		- dependency among tasks
		- certain tasks B depending on other tasks' A execution results need to be **blocked** before ready
		- tasks B need to be started by tasks A when it is ready

## Free-RTOS
### `cmsis_os2`
- abstraction layer for unifying the APIs provided by different RTOS
```c
// Exposed API to create a thread
osThreadId_t osThreadNew (osThreadFunc_t func, void *argument, const osThreadAttr_t *attr)

// Called API to create a thread (FreeRTOS)
xTaskCreate ((TaskFunction_t)func, name, (uint16_t)stack, argument, prio, &hTask)
```

### priority
- **ATTENTION**
	- priorities definition can be different
		- conversion between different priority definitions 

---
## Source Code
```c
osStatus_t osKernelInitialize (void) {
  osStatus_t stat;

  if (IS_IRQ()) {
    stat = osErrorISR;
  }
  else {
    if (KernelState == osKernelInactive) {
      #if defined(USE_FreeRTOS_HEAP_5)
        vPortDefineHeapRegions (xHeapRegions);
      #endif
      KernelState = osKernelReady;
      stat = osOK;
    } else {
      stat = osError;
    }
  }

  return (stat);
}

#define IS_IRQ()                  (IS_IRQ_MODE() || IS_IRQ_MASKED())
#define IS_IRQ_MODE()             (__get_IPSR() != 0U)
#define IS_IRQ_MASKED()           (__get_PRIMASK() != 0U)
```
- when initializing the OS kernel
	- there shouldn't be any ISR active
		- guaranteed by checking `IPSR` register
	- there shouldn't be any ISR masked
		- which means the initialization might be interrupted?
			- guaranteed by checking the `PRIMASK` register

## Memory Barrier
- [ ] #task modern compiler optimization
	- trick to avoid the code being optimized by the compiler
		- mostly avoid the execution orders being changed!
```c
/* Scheduler utilities. */
#define portYIELD()																\
{																				\
	/* Set a PendSV to request a context switch. */								\
	portNVIC_INT_CTRL_REG = portNVIC_PENDSVSET_BIT;								\
																				\
	/* Barriers are normally not required but do ensure the code is completely	\
	within the specified behaviour for the architecture. */						\
	__dsb( portSY_FULL_READ_WRITE );											\
	__isb( portSY_FULL_READ_WRITE );											\
}
/*-----------------------------------------------------------*/
```
- reference: [ARM Compiler toolchain Assembler Reference Version 4.1](https://developer.arm.com/documentation/dui0489/c/arm-and-thumb-instructions/miscellaneous-instructions/dmb--dsb--and-isb)

- [ ] #task critical sections
```c
/* Critical section management. */
extern void vPortEnterCritical( void );
extern void vPortExitCritical( void );

#define portDISABLE_INTERRUPTS()				vPortRaiseBASEPRI()
#define portENABLE_INTERRUPTS()					vPortSetBASEPRI( 0 )
#define portENTER_CRITICAL()					vPortEnterCritical()
#define portEXIT_CRITICAL()						vPortExitCritical()
#define portSET_INTERRUPT_MASK_FROM_ISR()		ulPortRaiseBASEPRI()
#define portCLEAR_INTERRUPT_MASK_FROM_ISR(x)	vPortSetBASEPRI(x)

/*-----------------------------------------------------------*/
```

- how to disable all the interrupts by increasing the `BASEPRI`
	- critical section
```c

static portFORCE_INLINE void vPortRaiseBASEPRI( void )
{
uint32_t ulNewBASEPRI = configMAX_SYSCALL_INTERRUPT_PRIORITY;

	__asm
	{
		/* Set BASEPRI to the max syscall priority to effect a critical
		section. */
		msr basepri, ulNewBASEPRI
		dsb
		isb
	}
}
/*-----------------------------------------------------------*/
```

## Heap
- memory allocation
	- header
		- position
		- size
	- required size of bytes

## Stack
- each function has its own stack
	- calling relationship
	- local variables
	- used to restore all the context when switching from another task

## `heap1.c`

```c
    /* Ensure that blocks are always aligned. */
    #if ( portBYTE_ALIGNMENT != 1 )
    {
        if( xWantedSize & portBYTE_ALIGNMENT_MASK )
        {
            /* Byte alignment required. Check for overflow. */
            if( ( xWantedSize + ( portBYTE_ALIGNMENT - ( xWantedSize & portBYTE_ALIGNMENT_MASK ) ) ) > xWantedSize )
            {
                xWantedSize += ( portBYTE_ALIGNMENT - ( xWantedSize & portBYTE_ALIGNMENT_MASK ) );
            }
            else
            {
                xWantedSize = 0;
            }
        }
    }
```
- allocated bytes are more than needed
	- `portBYTE_ALIGNMENT - ( xWantedSize & portBYTE_ALIGNMENT_MASK )`
		- always block aligned

## 可重入函数 re-entrant functions
- 函数在执行过程中，中断响应可能对于函数的执行结果产生影响
- 函数具有如下特点
	- 使用了静态变量
	- 不能修改函数本身
	- 调用了另一个不重入函数
- reference: https://www.geeksforgeeks.org/reentrant-function/