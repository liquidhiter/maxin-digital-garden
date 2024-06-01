---
{"dg-publish":true,"permalink":"/03-es-deep-dive/2-free-rtos/","noteIcon":"","created":"2024-03-09T22:13:22.289+01:00","updated":"2024-06-01T17:23:34.104+02:00"}
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


## FreeRTOS基本介绍
- 开源免费，开源协议使得商业使用也不用开源代码
- 市场占比超过20%
	- 中间件
		- FreeRTOS-FAT文件系统
		- FreeRTOS-TCP网络协议栈
### 基本概念
- 前后台系统
- 时间相关性很强的关键操作一定是靠中断服务来保证的
	- 中断响应的时间很快
		- ms级？
	- 中断服务提供的信息一直要等到后台程序运行到该处理这个信息时才能得到处理
		- 处理信息的及时性，称为`任务机响应时间`
		- 最坏情况下，循环的执行时间
			- 信息处理是last step？
	- 省电角度
		- 微处理器处于HALT状态
		- 一切都依靠中断服务来完成
- 代码的临界段
	- 不允许中断进入导致代码执行中断
- 资源
- 共享资源
	- 互斥
- 任务
	- 如何将一个应用程序分割成多个任务
	- 任务的状态
		- 休眠态
			- 任务驻留在内存中（何时以及如何拷贝？）
		- 就绪态
		- 运行态
		- 挂起态
			- 等待某一时间的发生？
		- 中断态
	- 任务切换
		- context switch
			- 保存CPU寄存器
				- 入栈操作
		- overhead，取决于CPU内部寄存器需要入栈的数量
- 内核
	- 管理任务，为每个任务分配CPU时间
	- 负责任务之间的通信
	- 任务切换
	- overhead
		- 服务执行时间
		- ROM代码空间
		- RAM内核数据结构运行时空间
		- 每个任务需要的栈空间
- 调度
	- scheduler 和 dispatcher
		- they shouldnt be the same, right?
	- 多数基于优先级来调度
- 不可剥夺型内核
	- 合作性多任务
	- 任务切换当且仅当运行的任务主动放弃CPU的使用权（运行结束）
	- 可以使用不可重入函数，因为任务不可被抢占
	- example![Z - assets/images/Pasted image 20240601161433.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601161433.png)
	- *高优先级的任务通过调用内核服务函数中的`延迟一个时钟节拍`来释放CPU的使用权*
	- 主要缺陷
		- 响应时间可能较长
			- 例如，上述例子中的，高优先级中断响应任务需要等到当前任务完成，并成功释放CPU的使用权后才能被内核做任务切换，从而被执行
- 可剥夺型内核
	- 主要的应用场景
		- 系统响应时间很重要
			- 最高优先级的任务一旦就绪，总能得到CPU的使用权
		- 频繁的任务切换造成的overhead ？
			- 非抢占式内核
				- 任务切换发生在CPU闲置时
			- 抢占式内核
				- 任务切换可能会发生在当前任务执行时
	- 优点
		- 任务响应时间得以最优化
			- 更高优先级的任务总是能被及时执行
			- 可预期的执行时间
	- 缺点
		- 必须满足互斥条件，才能调用可重入函数
- 可重入函数
	- 定义
		- 可重入函数
			- 可以被一个以上的任务调用，而不必担心数据会遭到破坏
			- 任何时刻都可以被中断，而相应的数据不会丢失
		- 局部变量
		- 全局变量，需予以保护
	- example![Z - assets/images/Pasted image 20240601170528.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601170528.png)
		- 中断发生在执行`swap`函数时
		- 低优先级任务中`y`中最终值为`3`
	- 如何使得函数具有可重入性
		- 避免使用全局变量
		- 通过关闭中断（critical section)来保护函数的执行
		- 使用信号量
- 时间片轮番调度法
	- 某个任务的运行执行一个事先确定的时间，称为时间额度(`quantum`)
	- 任务切换发生在
		- 当前任务已空闲
		- 当前任务在分配给其的时间片结束前已经完成了
		- 时间片已经结束
- 任务优先级
- 静态优先级
	- 应用程序在执行过程中优先级不变
- 动态优先级
	- 避免优先级反转
- 优先级反转
	- example![Z - assets/images/Pasted image 20240601172227.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601172227.png)
	- 用于更高优先级的任务，因为共享资源被更低优先级的任务占用，导致了其必须等到更低优先级的任务执行完以后，共享资源得以释放以后，才能得到执行
		- 在这种情况下，更高优先级的任务，实际上，降到了更低优先级
	- 解决方法
		- 在低任务优先级使用共享资源时，提升任务优先级，以至高于任何使用该资源的任务，从而确保共享资源能够尽快被释放
			- 改变任务的优先级需要很多时间？
		- 内核自动变换任务的优先级
			- 优先级继承
		- example![Z - assets/images/Pasted image 20240601172332.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601172332.png)