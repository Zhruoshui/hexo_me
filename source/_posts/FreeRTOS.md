---
title: FreeRTOS
abbrlink: 34230
date: 2024-07-10 15:14:06
tags:
  - 嵌入式操作系统
description: 本文用于记录嵌入式操作系统的学习过程
categories:
  - 嵌入式开发
cover: 'https://image.aruoshui.fun/i/2024/12/31/u1be33-0.webp'
swiper_index:
---


# 学习
## 堆
堆，heap，就是一块空闲的内存，需要提供管理函数
- malloc：从堆里划出一块空间给程序使用
- free：用完后，再把它标记为"空闲"的，可以再次使用s

## 栈
栈，stack，函数调用时局部变量保存在栈中，当前程序的环境也是保存在栈中
可以从堆中分配一块空间用作栈

## 源码结构
### 目录结构
![代码结构](https://s2.loli.net/2024/07/10/7f9cvzYARSsqjtx.png)

### 核心
FreeRTOS的最核心文件只有2个：
- FreeRTOS/Source/tasks.c
- FreeRTOS/Source/list.c

| FreeRTOS/source/下的文件 | 作用     | 
| ------------------------ | -------- | 
| task.c        | 必须，任务操作 | 
| list.c        | 必须，列表 | 
| queue.c        | 基本必须，提供队列操作、信号量操作  | 
| timer.c        | 可选，software timer  |
| event_groups.c        | 可选，提供event group功能  |
| croutine.c        | 可选，基本过时  |   

## 代码规范
![变量命名](https://s2.loli.net/2024/07/10/1lvz8NeSyaOxLT3.png)
![函数名](https://s2.loli.net/2024/07/10/WcDqbwXBUAxj9Ei.png)

## 创建任务函数详解
### 任务函数原形
任务就是一个函数，原型如下：
`void ATaskFunction( void *pvParameters );`

### 任务创建
在FreeRTOS中，任务可以通过静态创建和动态创建两种方式来实现。他们只有在任务创建的初期和能否释放栈有区别，最终使用是一模一样的。

静态创建的任务，栈是存放在数组里面的。因此静态创建任务的栈无法释放，在编译初期就定好了。
动态创建的任务，栈是通过类似malloc的函数实现的，也就是堆区域。堆是可以通过类似free函数释放的。

**栈的空间分配是重中之重**，关乎到每个任务函数内部使用的空间 


#### 两种创建方式如何抉择
1. 选择静态创建还是动态创建取决于应用的具体需求。如果你的任务具备以下特征，就推荐使用静态创建任务：
任务的数量和属性在编译时就确定，不需要动态创建或删除任务。
任务的栈空间需求可以预先估计，不需要动态调整。
堆空间有限，或者想要节省堆空间。
对任务创建的速度和可靠性有较高的要求。

2. 当你的任务具备以下特征，就推荐使用动态创建任务：
任务的数量和属性在运行时才确定，需要动态创建或删除任务。
任务的栈空间需求难以预先估计，需要动态调整。
堆空间充足，或者不在乎堆空间的占用。
对任务创建的速度和可靠性没有较高的要求。

#### 动态分配

```cpp
BaseType_t xTaskCreate( TaskFunction_t pxTaskCode,
                            const char * const pcName, /*lint !e971 Unqualified char types are allowed for strings and single characters only. */
                            const configSTACK_DEPTH_TYPE usStackDepth,
                            void * const pvParameters,                //参数
                            UBaseType_t uxPriority,                   //优先级
                            TaskHandle_t * const pxCreatedTask )      //任务控制块-> TCB的指针
```
![说明](https://s2.loli.net/2024/07/10/WKEtjmyd1gB2OMC.png)

例子：
```cpp
xTaskCreate(Task1Function, "Task1", 100, NULL, 1, &xHandleTask1);
xTaskCreate(Task2Function, "Task2", 100, NULL, 1, NULL);
```
#### 静态分配
TCB结构体需要实现分配好，栈也需要首先分配好
```cpp
    TaskHandle_t xTaskCreateStatic( TaskFunction_t pxTaskCode,
                                    const char * const pcName, /*lint !e971 Unqualified char types are allowed for strings and single characters only. */
                                    const uint32_t ulStackDepth,
                                    void * const pvParameters,
                                    UBaseType_t uxPriority,
                                    StackType_t * const puxStackBuffer,
                                    StaticTask_t * const pxTaskBuffer )
```

为了防止
```cpp
StackType_t xIdleTaskStack[100];
StaticTask_t xIdleTaskTCB;

void vApplicationGetIdleTaskMemory( StaticTask_t ** ppxIdleTaskTCBBuffer,
                                    StackType_t ** ppxIdleTaskStackBuffer,
                                    uint32_t * pulIdleTaskStackSize )
{
    *ppxIdleTaskTCBBuffer = &xIdleTaskTCB;
    *ppxIdleTaskStackBuffer = xIdleTaskStack;
    *pulIdleTaskStackSize = 100;
}
```


例子：
```C++
StackType_t xTask3Stack[100];   // 栈空间
StaticTask_t xTask3TCB;         // TCB任务块

xTaskCreateStatic(Task3Function, "Task3", 100, NULL, 1, xTask3Stack, &xTask3TCB);
```

### 任务优先级
freertos是数字越小优先级越低，优先级的取值范围是：0~(configMAX_PRIORITIES – 1)
#### tick中断
![一张图](https://s2.loli.net/2024/07/10/snU475bkLgiSIEq.png)



### 任务的删除
#### 函数原形
`void vTaskDelete( TaskHandle_t xTaskToDelete );`

传入的参数是任务句柄
使用xTaskCreate创建任务时可以得到一个句柄。也可传入NULL，这表示删除自己。

1. 自杀： vTaskDelete(NULL)
2. 被杀：别的任务执行 vTaskDelete(pvTaskCode) ，pvTaskCode是自己的句柄
3. 杀人：执行 vTaskDelete(pvTaskCode) ，pvTaskCode是别的任务的句柄、

{% note info flat %}
（1）任务自杀通常由任务自己主动发起，而删除其他任务是由系统中的某个任务请求删除另一个任务。
（2）在实际应用中，需要根据具体的需求和设计来选择使用哪种方法。在任何情况下，都需要确保在删除任务之前，已经合理地释放了任务占用的资源，以避免资源泄漏和系统不稳定性。删除任务释放的资源要考虑以下内容：

1. 避免删除正在执行的任务：尽量避免删除正在执行的任务，因为这可能导致未定义的行为。通常，应该在目标任务主动结束执行或者在任务的代码中检查某些条件后再请求删除。
2. 处理资源释放：确保在删除任务之前释放任务使用的资源。这包括释放动态分配的内存、关闭文件句柄、释放占用的硬件资源等。如果任务在删除时仍然占用资源，可能会导致资源泄漏或系统不稳定。
3. 处理同步和互斥：如果目标任务与其他任务之间存在同步或互斥关系，确保在删除任务之前解除这些关系，以免引起竞态条件或死锁。
4. 避免删除空闲任务：在 FreeRTOS 中，空闲任务（Idle Task）用于在系统没有其他任务需要执行时运行。删除空闲任务可能导致系统无法正常工作，应该谨慎使用。例如，任务删除之后的堆栈释放，是在空闲任务中执行，当空闲任务删除之后，堆栈将需要通过其他任务手动释放，这样将会增加工作量。
{% endnote %}
   
### 任务状态
学习过操作系统就很简单，直接看例子:
```cpp
void Task1Function(void * param)
{
	TickType_t tStart = xTaskGetTickCount();
	TickType_t t;
	int flag = 0;
	
	while (1)
	{
		t = xTaskGetTickCount();
		
		task1flagrun = 1;
		task2flagrun = 0;
		task3flagrun = 0;
		printf("1");

		if (!flag && (t > tStart + 10))
		{
			vTaskSuspend(xHandleTask3);
			flag = 1;
		}

		if (t > tStart + 20)
		{
			vTaskResume(xHandleTask3);
		}
	}
}

void Task2Function(void * param)
{
	while (1)
	{
		task1flagrun = 0;
		task2flagrun = 1;
		task3flagrun = 0;
		printf("2");

		vTaskDelay(10);
	}
}

void Task3Function(void * param)
{
	while (1)
	{
		task1flagrun = 0;
		task2flagrun = 0; 
		task3flagrun = 1;
		printf("3");
	}
}

```
![结果](https://s2.loli.net/2024/07/10/knL9HAC1QbuE2Rc.png)

### 空闲任务及钩子函数
看这个例子：
```cpp
void Task1Function(void * param)
{
	TaskHandle_t xHandleTask2;
	BaseType_t xReturn;
	
	while (1)
	{
		task1flagrun = 1;
		task2flagrun = 0;
		taskidleflagrun = 0;
		printf("1");
		xReturn = xTaskCreate(Task2Function, "Task2", 1024, NULL, 2, &xHandleTask2);
		if (xReturn != pdPASS)
			printf("xTaskCreate err\r\n");
		//vTaskDelete(xHandleTask2);
			
	}
}

void Task2Function(void * param)
{
	while (1)
	{
		task1flagrun = 0;
		task2flagrun = 1;
		taskidleflagrun = 0;
		printf("2");
		//vTaskDelay(2);
		vTaskDelete(NULL);
	}
}
```
任务1会反复创建任务2，任务2执行自杀，不能完成清理尸体，需要空闲任务进行清空(自杀由空闲任务处理实体，他杀由杀者处理实体)


空闲任务(Idle任务)的作用：释放被删除的任务的内存。
一个良好的程序，它的任务都是事件驱动的：平时大部分时间处于阻塞状态。有可能我们自己创建的所有任务都无法执行，但是调度器必须能找到一个可以运
行的任务：所以，我们要提供空闲任务。在使用 vTaskStartScheduler() 函数来创建、启动调度器时，这个函数内部会创建空闲任务：
1. 空闲任务优先级为0：它不能阻碍用户任务运行
2. 空闲任务要么处于就绪态，要么处于运行态，永远不会阻塞
3. 空闲任务的优先级为0，这以为着一旦某个用户的任务变为就绪态，那么空闲任务马上被切换出去，让这个用户任务运行。在这种情况下，我们说用户任务"抢占"(pre-empt)了空闲任务，这是由调度器实现的。

**如果使用 vTaskDelete() 来删除任务，那么你就要确保空闲任务有机会执行，否则就无法释放被删除任务的内存**

#### 钩子
我们可以添加一个空闲任务的钩子函数(Idle Task Hook Functions)，空闲任务的循环没执行一次，就会调用一次钩子函数。 

钩子函数的作用有这些：
1. 执行一些低优先级的、后台的、需要连续执行的函数测量系统的空闲时间：
2. 空闲任务能被执行就意味着所有的高优先级任务都停止了，所以测量空闲任务占据的时间，就可以算出处理器占用率。
3. 让系统进入省电模式：空闲任务能被执行就意味着没有重要的事情要做，当然可以进入省电模式


空闲任务的钩子函数的限制：
1. 不能导致空闲任务进入阻塞状态、暂停状态
2. 如果你会使用 vTaskDelete() 来删除任务，那么钩子函数要非常高效地执行。如果空闲任务移植
3. 卡在钩子函数里的话，它就无法释放内存。

## delay函数
有两个Delay函数：
- vTaskDelay：至少等待指定个数的Tick Interrupt才能变为就绪状态
- vTaskDelayUntil：等待到指定的绝对时刻，才能变为就绪态。

![说明](https://s2.loli.net/2024/07/10/2HDKAez7MGtjvTF.png)


## 任务调度算法
### 配置调度算法
通过配置文件FreeRTOSConfig.h的两个配置项来配置调度算法：configUSE_PREEMPTION、configUSE_TIME_SLICING
```cpp
#define configUSE_PREEMPTION		1    // 支持抢占
#define configUSE_TIME_SLICING      1    // 同优先级的任务交替执行
#define configIDLE_SHOULD_YIELD		1    // 空闲任务让步
```
### 抢占
- 抢占时：高优先级任务就绪时，就可以马上执行
- 不抢占时：优先级失去意义了，既然不能抢占就只能协商了，一个任务执行，其他任务都无法执行。即使其他任务已经超时、即使它的优先级更高，都没办法执行。

### 时间片
- 时间片轮转：在Tick中断中会引起任务切换
- 时间片不轮转：高优先级任务就绪时会引起任务切换，高优先级任务不再运行时也会引起任务切换。

### 空闲任务让步
让步时：假如设置空闲任务为无限循环，则在空闲任务的每个循环中，会主动让出处理器
不让步时：空闲任务跟其他任务同等待遇


## 同步互斥与通信概述
能实现同步、互斥的内核方法有：任务通知(task notification)、队列(queue)、事件组(event group)、信号量(semaphoe)、互斥量(mutex)。

它们都有类似的操作方法：获取/释放、阻塞/唤醒、超时。比如：
A获取资源，用完后A释放资源
A获取不到资源则阻塞，B释放资源并把A唤醒
A获取不到资源则阻塞，并定个闹钟；A要么超时返回，要么在这段时间内因为B释放资源而被唤醒。

![说明](https://s2.loli.net/2024/07/11/l8GZUzmsYNAq9H5.png)

## 队列
队列其实是一个环形缓冲区，定义如下：
```cpp
typedef struct QueueDefinition /* The old naming convention is used to prevent breaking kernel aware debuggers. */
{
    int8_t * pcHead;           /*< Points to the beginning of the queue storage area. */
    int8_t * pcWriteTo;        /*< Points to the free next place in the storage area. */

    union
    {
        QueuePointers_t xQueue;     /*< Data required exclusively when this structure is used as a queue. */
        SemaphoreData_t xSemaphore; /*< Data required exclusively when this structure is used as a semaphore. */
    } u;

    List_t xTasksWaitingToSend;             /*< List of tasks that are blocked waiting to post onto this queue.  Stored in priority order. */
    List_t xTasksWaitingToReceive;          /*< List of tasks that are blocked waiting to read from this queue.  Stored in priority order. */

    volatile UBaseType_t uxMessagesWaiting; /*< The number of items currently in the queue. */
    UBaseType_t uxLength;                   /*< The length of the queue defined as the number of items it will hold, not the number of bytes. */
    UBaseType_t uxItemSize;                 /*< The size of each items that the queue will hold. */

    volatile int8_t cRxLock;                /*< Stores the number of items received from the queue (removed from the queue) while the queue was locked.  Set to queueUNLOCKED when the queue is not locked. */
    volatile int8_t cTxLock;                /*< Stores the number of items transmitted to the queue (added to the queue) while the queue was locked.  Set to queueUNLOCKED when the queue is not locked. */

    #if ( ( configSUPPORT_STATIC_ALLOCATION == 1 ) && ( configSUPPORT_DYNAMIC_ALLOCATION == 1 ) )
        uint8_t ucStaticallyAllocated; /*< Set to pdTRUE if the memory used by the queue was statically allocated to ensure no attempt is made to free the memory. */
    #endif

    #if ( configUSE_QUEUE_SETS == 1 )
        struct QueueDefinition * pxQueueSetContainer;
    #endif

    #if ( configUSE_TRACE_FACILITY == 1 )
        UBaseType_t uxQueueNumber;
        uint8_t ucQueueType;
    #endif
} xQUEUE;

```
使用队列的流程：创建队列、写队列、读队列、删除队列。

### 创建
![动态创建队列](https://s2.loli.net/2024/07/11/tdR6ZbWU5TAsPGg.png)
![静态创建队列](https://s2.loli.net/2024/07/11/9HQwarTkceNI5Ud.png)

### 写
```cpp
/* 等同于xQueueSendToBack
* 往队列尾部写入数据，如果没有空间，阻塞时间为xTicksToWait
*/
BaseType_t xQueueSend(
QueueHandle_t xQueue,
const void *pvItemToQueue,
TickType_t xTicksToWait
);
/*
* 往队列尾部写入数据，如果没有空间，阻塞时间为xTicksToWait
*/
BaseType_t xQueueSendToBack(
QueueHandle_t xQueue,
const void *pvItemToQueue,
TickType_t xTicksToWait
);
/*
* 往队列尾部写入数据，此函数可以在中断函数中使用，不可阻塞
*/
BaseType_t xQueueSendToBackFromISR(
QueueHandle_t xQueue,
const void *pvItemToQueue,
BaseType_t *pxHigherPriorityTaskWoken
);
/*
* 往队列头部写入数据，如果没有空间，阻塞时间为xTicksToWait
*/
BaseType_t xQueueSendToFront(
QueueHandle_t xQueue,
const void *pvItemToQueue,
TickType_t xTicksToWait
);
/*
* 往队列头部写入数据，此函数可以在中断函数中使用，不可阻塞
*/
BaseType_t xQueueSendToFrontFromISR(
QueueHandle_t xQueue,
const void *pvItemToQueue,
BaseType_t *pxHigherPriorityTaskWoken
);
```

![写](https://s2.loli.net/2024/07/11/d9ctCQiL7PY6upg.png)


### 读
使用 xQueueReceive() 函数读队列，读到一个数据后，队列中该数据会被移除。
```cpp
BaseType_t xQueueReceive( QueueHandle_t xQueue,
void * const pvBuffer,
TickType_t xTicksToWait );

BaseType_t xQueueReceiveFromISR(
QueueHandle_t xQueue,
void *pvBuffer,
BaseType_t *pxTaskWoken
);
```
![读](https://s2.loli.net/2024/07/11/1iSsAB3rMFQqLoT.png)

### 删除队列
删除队列的函数为 vQueueDelete() ，只能删除使用动态方法创建的队列，它会释放内存。

### 复位
队列刚被创建时，里面没有数据；使用过程中可以调用 xQueueReset() 把队列恢复为初始状态

## 信号量
只需要传递状态，并不需要传递具体的信息，这个时候就可以使用信号量

### 两种信号量
信号量的计数值都有限制：限定了最大值。如果最大值被限定为1，那么它就是二进制信号量；
如果最大值不是1，它就是计数型信号量。

### 信号量创建
使用信号量之前，要先创建，得到一个句柄；使用信号量时，要使用句柄来表明使用哪个信号量。
对于二进制信号量、计数型信号量，它们的创建函数不一样：

二进制信号量：
```cpp
/* 创建一个二进制信号量，返回它的句柄。
* 此函数内部会分配信号量结构体
* 返回值: 返回句柄，非NULL表示成功
*/
SemaphoreHandle_t xSemaphoreCreateBinary( void );
/* 创建一个二进制信号量，返回它的句柄。
* 此函数无需动态分配内存，所以需要先有一个StaticSemaphore_t结构体，并传入它的指针
* 返回值: 返回句柄，非NULL表示成功
*/
SemaphoreHandle_t xSemaphoreCreateBinaryStatic( StaticSemaphore_t *pxSemaphoreBuffer );
```

计数型信号量:
```cpp
/* 创建一个计数型信号量，返回它的句柄。
* 此函数内部会分配信号量结构体
* uxMaxCount: 最大计数值
* uxInitialCount: 初始计数值
* 返回值: 返回句柄，非NULL表示成功
*/
SemaphoreHandle_t xSemaphoreCreateCounting(UBaseType_t uxMaxCount, UBaseType_t
uxInitialCount);
/* 创建一个计数型信号量，返回它的句柄。
* 此函数无需动态分配内存，所以需要先有一个StaticSemaphore_t结构体，并传入它的指针
* uxMaxCount: 最大计数值
* uxInitialCount: 初始计数值
* pxSemaphoreBuffer: StaticSemaphore_t结构体指针
* 返回值: 返回句柄，非NULL表示成功
*/
SemaphoreHandle_t xSemaphoreCreateCountingStatic( UBaseType_t uxMaxCount,
                                                    UBaseType_t uxInitialCount,
                                                    StaticSemaphore_t
*pxSemaphoreBuffer );

```

### 信号量的删除
对于动态创建的信号量，不再需要它们时，可以删除它们以回收内存。
```cpp
/*
* xSemaphore: 信号量句柄，你要删除哪个信号量
*/
void vSemaphoreDelete( SemaphoreHandle_t xSemaphore );
```

### give
释放信号量
需要注意，信号量的释放都是立即返回的。只有获取信号量才会进行阻塞等待。
```cpp
/**
 * @brief  释放信号量
 *
 * @param  xSemaphore 要释放的信号量句柄
 *
 * @return  释放成功，返回pdTRUE。否则返回 pdFALSE。
 */
BaseType_t xSemaphoreGive( SemaphoreHandle_t xSemaphore );
```

### take
获取信号量
```cpp
/**
 * @brief  获取信号量
 *
 * @param  xSemaphore   要获取的信号量句柄
 *        -xTicksToWait 超时时间，0 表示立即返回，portMAX_DELAY表示无限等待, 一直阻塞直到成功
 *
 * @return  成功获取信号量，返回pdPASS。否则返回pdFALSE
 */
BaseType_t xSemaphoreTake( SemaphoreHandle_t xSemaphore,TickType_t xTicksToWait );

```

## 互斥量
互斥量能够解决优先级反转问题，而信号量不行。

互斥量也被称为互斥锁，使用过程如下：
互斥量初始值为1
任务A想访问临界资源，先获得并占有互斥量，然后开始访问
任务B也想访问临界资源，也要先获得互斥量：被别人占有了，于是阻塞
任务A使用完毕，释放互斥量；任务B被唤醒、得到并占有互斥量，然后开始访问临界资源
任务B使用完毕，释放互斥量

### 互斥量的创建
使用互斥量时，先创建、然后去获得、释放它。使用句柄来表示一个互斥量。各类操作函数，比如删除、give/take，跟一般是信号量是一样的。但是要注意互斥量不能在ISR中使用

```cpp
/* 创建一个互斥量，返回它的句柄。
* 此函数内部会分配互斥量结构体
* 返回值: 返回句柄，非NULL表示成功
*/
SemaphoreHandle_t xSemaphoreCreateMutex( void );
/* 创建一个互斥量，返回它的句柄。
* 此函数无需动态分配内存，所以需要先有一个StaticSemaphore_t结构体，并传入它的指针
* 返回值: 返回句柄，非NULL表示成功
*/
SemaphoreHandle_t xSemaphoreCreateMutexStatic( StaticSemaphore_t *pxMutexBuffer
);
```
使用互斥量，需要在配置文件FreeRTOSConfig.h中定义：
`##define configUSE_MUTEXES 1`




## 队列集
队列只能存放同一种类型的数据，那如果我们需要在任务之间传输不同类型的消息，应该如何处理呢？例如遥控车，可能是按键式的遥控器（传输的是整型数据），也可能会兼容遥感式的遥控器（传输的是浮点数据）。这两种遥控器都能够控制小车，那么应当如何进行数据传输呢？

**队列集的本质也是队列，只不过里面存放的是队列句柄，而不是数据。**
```cpp
QueueSetHandle_t xQueueCreateSet( const UBaseType_t uxEventQueueLength )
{
QueueSetHandle_t pxQueue;

	pxQueue = xQueueGenericCreate( uxEventQueueLength, ( UBaseType_t ) sizeof( Queue_t * ), queueQUEUE_TYPE_SET );

	return pxQueue;
}
```


## 事件组
事件组可以用下面的图进行解释
![事件组原理](https://s2.loli.net/2024/07/18/mYKQapgxXolw7Zq.png)
其创建和删除跟之前的互斥等类型相同，都是以handle为依据，主要是事件组的设置和等待事件

### 设置事件
可以设置事件组的某个位、某些位，使用的函数有2个：
在任务中使用 xEventGroupSetBits()
在ISR中使用 xEventGroupSetBitsFromISR()
有一个或多个任务在等待事件，如果这些事件符合这些任务的期望，那么任务还会被唤醒。

```cpp
/* 设置事件组中的位
* xEventGroup: 哪个事件组
* uxBitsToSet: 设置哪些位?
* 如果uxBitsToSet的bitX, bitY为1, 那么事件组中的bitX, bitY被设置为1
* 可以用来设置多个位，比如 0x15 就表示设置bit4, bit2, bit0
* 返回值: 返回原来的事件值(没什么意义, 因为很可能已经被其他任务修改了)
*/
EventBits_t xEventGroupSetBits( EventGroupHandle_t xEventGroup,const EventBits_t uxBitsToSet );
/* 设置事件组中的位
* xEventGroup: 哪个事件组
* uxBitsToSet: 设置哪些位?
* 如果uxBitsToSet的bitX, bitY为1, 那么事件组中的bitX, bitY被设置为1
* 可以用来设置多个位，比如 0x15 就表示设置bit4, bit2, bit0
* pxHigherPriorityTaskWoken: 有没有导致更高优先级的任务进入就绪态? pdTRUE-有,
pdFALSE-没有
* 返回值: pdPASS-成功, pdFALSE-失败
*/
BaseType_t xEventGroupSetBitsFromISR( EventGroupHandle_t xEventGroup,const EventBits_t uxBitsToSet, BaseType_t * pxHigherPriorityTaskWoken );
```


### 等待事件
使用 xEventGroupWaitBits 来等待事件，可以等待某一位、某些位中的任意一个，也可以等待多位；
等到期望的事件后，还可以清除某些位。
```cpp
EventBits_t xEventGroupWaitBits( EventGroupHandle_t xEventGroup,
        const EventBits_t uxBitsToWaitFor,
        const BaseType_t xClearOnExit,
        const BaseType_t xWaitForAllBits,
        TickType_t xTicksToWait );
```
![说明](https://s2.loli.net/2024/07/18/94Xf5Cdvjnth3r2.png)
可以使用 xEventGroupWaitBits() 等待期望的事件，它发生之后再使用 xEventGroupClearBits()
来清除。但是这两个函数之间，有可能被其他任务或中断抢占，它们可能会修改事件组。

可以使用设置 xClearOnExit 为pdTRUE，使得对事件组的测试、清零都在 xEventGroupWaitBits()
函数内部完成，这是一个原子操作。


### 同步点
{% note info flat %}
有一个事情需要多个任务协同，比如：
    任务A：炒菜
    任务B：买酒
    任务C：摆台
A、B、C做好自己的事后，还要等别人做完；大家一起做完，才可开饭
{% endnote %}

使用 xEventGroupSync() 函数可以同步多个任务：
可以设置某位、某些位，表示自己做了什么事。
可以等待某位、某些位，表示要等等其他任务。
期望的时间发生后， xEventGroupSync() 才会成功返回。
xEventGroupSync 成功返回后，会清除事件。

```cpp
EventBits_t xEventGroupSync( EventGroupHandle_t xEventGroup,
                        const EventBits_t uxBitsToSet,
                        const EventBits_t uxBitsToWaitFor,
                        TickType_t xTicksToWait );
```
![说明](https://s2.loli.net/2024/07/18/vQXBRWighezVK2L.png)


## 任务通知

使用队列、信号量、事件组等等方法时，并不知道对方是谁。使用任务通知时，可以明确指定：通知哪个任务。
使用队列、信号量、事件组时，我们都要事先创建对应的结构体，双方通过中间的结构体通信：使用任务通知时，任务结构体TCB中就包含了内部对象，可以直接接收别人发过来的"通知"：

任务通知的优势：
效率更高：使用任务通知来发送事件、数据给某个任务时，效率更高。比队列、信号量、事件组都
有大的优势。
更节省内存：使用其他方法时都要先创建对应的结构体，使用任务通知时无需额外创建结构体。

### 通知状态和通知值
每个任务都有一个结构体：TCB(Task Control Block)，里面有2个成员：
一个是uint8_t类型，用来表示通知状态
一个是uint32_t类型，用来表示通知值

```cpp
typedef struct tskTaskControlBlock
{
......
/* configTASK_NOTIFICATION_ARRAY_ENTRIES = 1 */
volatile uint32_t ulNotifiedValue[ configTASK_NOTIFICATION_ARRAY_ENTRIES ];
volatile uint8_t ucNotifyState[ configTASK_NOTIFICATION_ARRAY_ENTRIES ];
......
} tskTCB;
```
- taskNOT_WAITING_NOTIFICATION：任务没有在等待通知
- taskWAITING_NOTIFICATION：任务在等待通知
- taskNOTIFICATION_RECEIVED：任务接收到了通知，也被称为pending(有数据了，待处理)





# 开发