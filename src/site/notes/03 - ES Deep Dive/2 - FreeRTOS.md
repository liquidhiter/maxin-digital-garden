---
{"dg-publish":true,"permalink":"/03-es-deep-dive/2-free-rtos/","noteIcon":"","created":"2024-03-09T22:13:22.289+01:00","updated":"2024-06-01T22:13:03.723+02:00"}
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
	- example
	 ![Z - assets/images/Pasted image 20240601161433.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601161433.png)
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
	- example
	 ![Z - assets/images/Pasted image 20240601170528.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601170528.png)
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
	- example
	 ![Z - assets/images/Pasted image 20240601172227.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601172227.png)
	- 用于更高优先级的任务，因为共享资源被更低优先级的任务占用，导致了其必须等到更低优先级的任务执行完以后，共享资源得以释放以后，才能得到执行
		- 在这种情况下，更高优先级的任务，实际上，降到了更低优先级
	- 解决方法
		- 在低任务优先级使用共享资源时，提升任务优先级，以至高于任何使用该资源的任务，从而确保共享资源能够尽快被释放
			- 改变任务的优先级需要很多时间？
		- 内核自动变换任务的优先级
			- 优先级继承
		- example
		 ![Z - assets/images/Pasted image 20240601172332.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601172332.png)
- 任务优先级分配
	- 综合软实时和硬实时
	- 单调执行率调度法 RMS(`Rate Monotonic Scheduling`)用于分配任务优先级
		- 基于任务执行的次数，执行最频繁的任务优先级最高
		- assumption
			- 任务都是周期性的
			- 任务之间不需要同步，没有共享资源或者数据交换等
			- 必须使用可剥夺型调度法
		- 所有时间敏感性任务占用CPU的总时间应小于`70%`，如果要求所有任务都满足硬实时条件
	- 系统设计原则
		- CPU的利用率应小于`60% ~ 70%`
	- 实际中，最高执行率的任务并非是最重要的任务
- 互斥条件
	- 任务间通信
		- 使用共享数据结构，特别是所有的任务都在一个单一的地址空间下
	- 方法
		- 关闭中断
			- 中断保护的代码需要足够简单，否则中断响应时间受到影响
				- 关中断时间不能超过内核本身的关中断时间
					- 数据由厂商提供
			- 在中断服务routine中处理共享变量的唯一方法
		- 测试并置位TAS(Test-and-Set)
			- TAS操作必须是一条不会被中断的指令
				- 否则需要先关闭中断再做TAS操作
					- 考虑到TAS保护的代码执行时间较长，使用中断保护不合适的
			- 某些处理器具有硬件TAS指令
		- 禁止，然后允许任务切换
			- 前提：任务和中断响应程序不共享变量
			- 不推荐使用，因为这有悖于调度器的设计原则
		- 信号量，`Semaphore`
			- 控制共享资源的使用权
			- 标志某个事件的发生
			- 使2个任务的行为同步
			- 类型
				- binary
					- 0 或 1
				- counting
			- 3中常规操作
				- `create`
				- `wait`
					- 等待超时
				- `signal`
					- 没有其它的任务等待信号量
						- 信号量的值简单地加1
					- 有其它的任务等待该信号量
						- 信号量的值不加1
						- 内核调度，切换某个任务进入就绪态
							- 等待信号量的所有任务中优先级最高的
							- 最早开始等待该信号量的任务，FIFO原则
						- 就绪态的任务有高于当前任务的优先级
							- 内核进行任务切换（可抢占式内核）
			- 使用信号量保护共享数据不会增加中断延迟时间
				- 中断开启，高优先级任务运行抢占
			- 使用场景
				- 使用共享（硬件）资源
					- 将信号量的创建和释放隐藏在驱动函数中
						- 资源的consumer并不需要知道信号量的存在
							- 只需要保证在其任务完成前，可以独占这一资源？
			- example
			 ![Z - assets/images/Pasted image 20240601203244.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601203244.png)
			- 计数式信号量
				- 某一资源可以同时为几个任务所用
				- example
					- 信号量管理缓冲区阵列(buffer pool) 
					 ![Z - assets/images/Pasted image 20240601203718.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601203718.png)
			- 避免过度的使用信号量
				- 申请和释放信号量会带来额外的overhead
				- *关闭中断可能是一种更高效的方式*
				- 个人理解
					- 如果对共享资源进行保护，其操作非常耗时间，使用信号量，避免延迟中断响应
				- example
					- 整数共享变量，关闭中断即可
					- 浮点数共享变量，使用信号量
						- 浮点运算时间较长（没有硬件浮点数处理器）
		- 死锁（deadlock）
			- 2个任务无限期地互相等待对方控制的资源
			- 解决方案
				- 内核允许用户在申请信号量时定义等待超时
		- 同步
			- 信号量，初始化为`0`
				- 某个任务与中断服务同步，或者与另一个任务同步，但是*这2个任务间没有数据交换*
				- 单向同步（unilateral rendezvous）
				 ![Z - assets/images/Pasted image 20240601212143.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601212143.png)
			- 计数式信号量
				- 信号量的值表示尚未得到处理的事件数
			- 多个任务等待同一个事件的发生
				- 优先级最高的任务优先
				- 最先开始等待事件的任务（FIFO原则）
			- 2个任务可以用2个信号量同步行为
				- 双向同步（bilateral rendezvous）
				 ![Z - assets/images/Pasted image 20240601212154.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601212154.png)
				- 注意：双向同步不可能发生在任务与ISR间，ISR不可能等待一个信号量！
		- 事件标志
			- 每个时间占1位（bit）
				- 任务或中断服务可以给某一位置位或者复位
		 ![Z - assets/images/Pasted image 20240601212516.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601212516.png)
			- 独立性同步
				- 任务与任何时间之一同步
			- 关联性同步
				- 任务与若干事件发生了同步
	- 任务间通信 （inter task communication）
		-  全局变量
			- *任务和中断服务程序通信的唯一方式*
				- 任务不知道什么时候这一变量被中断服务程序修改了
			- 发消息给另一个任务
	- 消息邮箱（message mail box）
		- 指针型变量？
		- 每个邮箱有相应的正在等待消息的任务列表
		- 内核允许定义等待超时
			- 超时后，该任务进入就绪态，并返回错误信息，报告等待超时错误
		-  消息放入邮箱后，会被传递给
			- 等待消息的任务中优先级最高的
			- 最开始等待信息的任务
		- 邮箱也可以作为binary semaphore使用
	- 消息队列（message queue）
		- 先进入消息队列的任务先传给任务
			- 支持FIFO和LIFO
		- 工作原理类似消息邮箱
		- [ ] 和消息邮箱相比，只是underlying的数据结构不同？
	- 中断
		- 通知CPU异步事件的发生
		- CPU保存现场，部分或者全部寄存器的值，再跳转到中断服务子程序
		- 关中断时间太长，可能会导致中断丢失
			- WHY？中断不是写入到寄存器中对应的位吗？
	- 中断延迟
		- *实时内核最重要的指标就是关中断的时间的长短*
			- *关中断的最长时间+开始执行中断服务子程序第1条指令的时间*
	- 中断响应
		- *从中断发生到开始执行用户的中断服务子程序来处理这个中断的时间*
			- *中断延迟 + 保存现场所需要的时间*
				- 不可抢占型
				- 可抢占型
					- 需要调用一个特定的函数，告知内核即将开始中断服务，使得内核可以跟踪中断的嵌套
						- + *内核进入中断服务函数的执行时间*
		- 考虑最坏情况下的响应中断的时间
	- 中断恢复时间
		- 处理器返回到被中断了的程序代码所需要的时间
			- *恢复CPU内部寄存器的时间 + 执行中断返回指令的时间*
		- 抢占型内核，还需要调用内核函数来判断中断是否脱离了所有的中断嵌套
			- *+ 判断是否有优先级更高的任务进入了就绪态的时间*
		![Z - assets/images/Pasted image 20240601215344.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240601215344.png)
	- 中断处理时间
		- 在中断服务中，通过信号量、邮箱或者消息队列通知一个任务是需要一定时间的
			- 如果事件处理需要的事件 < 给一个任务通知的时间
				- 在中断服务程序中处理这一事件
				- 开中断允许更高优先级的中断抢占
	- 非屏蔽中断（NMI）
		- 可以用于时间要求最严苛的中断服务
		- 使用NMI时，不能使用内核提供的服务，因为NMI无法关闭以处理临界区代码
		- 可以使用全局变量（atmoic）后来传递参数给NMI或者从NMI读取参数
		- 使用NMI产生普通的可屏蔽中断
	- 时钟节拍（clock tick）
		- 特定的周期性中断
			- 10 ~ 200 ms
		- 时钟节拍频率越快，系统的额外开销越大
			- 产生的中断越频繁
		- 实时的内核有将任务延时若干个时钟节拍的功能