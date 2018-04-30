# lab6 report

## 练习1: 使用 Round Robin 调度算法

1. 请理解并分析sched_class中各个函数指针的用法，并接合Round Robin调度算法描述ucore的调度执行过程

- init函数初始化调度队列
- enqueue函数将就绪进程加入调度队列
- dequeue函数将进程从调度队列中移除
- pick_next函数获取被选出调入执行的进程
- proc_tick函数处理时钟信息

Round Robin 调度算法是将所有的进程加入到一个队列中，每个进程都拥有相同的运行时间。每次调度器都会从队列中取出一个进程，如果进程在分配的运行时间内没有完成，则需要放弃CPU资源进入队列尾部，等待重新分配运行时间。

ucore调度的执行过程主要是：在内核初始化时调用sched_init函数选择调度器并对调度器初始化。每当进程被唤醒则调用sched_class_enqueue函数将其加入调度器等待调度的进程队列。每当发生进程切换时，首先调用sched_class_enqueue将当前正在运行的进程加入等待调度的进程队列，然后调用sched_class_pick_next函数获取接下来将执行的进程，并调用sched_class_dequeue将被选中即将执行的进程从调度队列中移除。每当产生时钟中断时，需要调用sched_class_proc_tick函数来更新调度器的时钟信息。

2. 请在实验报告中简要说明如何设计实现"多级反馈队列调度算法"，给出概要设计

多级反馈队列调度算法维护多个队列，每个新的进程加入第一个队列，当需要选择一个进程调入执行时，从第一个队列开始向后查找，遇到某个队列非空，那么从这个队列中取出一个进程调入执行。如果从某个队列调入的进程在时间片用完之后仍然没有结束，则将这个进程加入其调入时所在队列之后的一个队列，并且时间片加倍。

为了支持多个队列，首先需要在run_queue结构中增加多个队列的声明，并且在proc_struct结构中为进程添加队列号属性。算法的具体实现见multi_sched.c和multi_sched.h文件。


## 练习2: 实现 Stride Scheduling 调度算法

Stride Scheduling算法用一个小顶堆选择即将调入CPU执行的进程。初始化时，将堆设为空，进程数为0。出现新建进程时，将进程加入堆，并设置时间片和指向堆的指针，增加进程计数。当一个进程执行完毕后将其从堆中移除并减少进程计数。当需要调度时，从堆顶取出当前stride值最小的进程调入CPU执行，并将其stride增加pass大小，如果堆顶为空，则返回空指针。产生时钟中断的处理与其他调度算法相同，减少当前进程的执行时间，若剩余时间为0则进行调度。

具体实现见default_sched_stride.c


## 扩展练习 Challenge :实现 Linux 的 CFS 调度算法

CFS算法是想要让每个进程的运行时间尽可能相同，那么记录每个进程已经运行的时间即可。在进程控制块结构中增加一个属性表示运行时间。每次需要调度的时候，选择已经运行时间最少的进程调入CPU执行。为了快速选择出已经运行时间最少的进程，用小顶堆来维护各个进程的调入次序，即以已经运行的时间多少作为优先级。为了完成对运行时间的记录，在run_queue结构中增加一个小顶堆，并在进程控制块中增加已经运行的时间，优先级和小顶堆三个成员变量。优先级属性的数值越大，每个时钟中断产生时正在运行的进程所增加的运行时间就越多，即其实际运行时间越短。此外，当当前运行时间最短的进程需要重新调度时，必须适当增加其运行时间比如一个时间片的时间，以便于调入进程的选择。算法的初始化，进队和出队都和Stride Schedule算法相似，不同在于当产生时钟中断时需要增加当前进程已经运行的时间。具体实现见cfs_sched.c和cfs_sched.h文件。

对不同的调度算法进行测试需要在sched.c文件中的sched_init函数中选择对应调度算法类进行实例化。在三行不同的调度算法中选择其中两个注释掉即可。