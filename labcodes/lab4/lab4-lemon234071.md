与参考答案对比：  
练习一：  

	我没有初始化context  
练习二：  

	1、函数执行失败goto错地方  
	2、没有设置父指针 
	3、没用加local_intr_save（锁？还是什么东西）  

本实验中重要知识点：  

	*进程控制块  
	*进程的创建  
	*进程的资源  
	*进程状态  
	*进程管理调度  

很重要但未对应的知识点：

	*进程状态转换

练习一：

请在实验报告中简要说明你的设计实现过程。  
	根据实验指导书，以及注释初始化即可。
请说明proc_struct中 struct context context 和 struct trapframe *tf 成员变量含义和在本实验中的作用是啥？  

	context：进程的上下文，用于进程切换（参见switch.S）
	本实验在copy_thread中设置了initproc的执行现场中主要的两个信息：上次停止执行时的下一条指令地址context.eip和上次停止执行时的堆栈地址context.esp。

	trapframe *tf：中断帧的指针，总是指向内核栈的某个位置。
	用来支持用户态内核态的切换，以及中断嵌套。

练习二：

请在实验报告中简要说明你的设计实现过程。  

	根据注释和指导书完成代码，make qemu发生缺页异常，发现在插入hash链表前需要分配pid。
请说明ucore是否做到给每个新fork的线程一个唯一的id？请说明你的分析和理由。
	
	根据get_pid，emmm...有点没看懂。
练习三： 

请在实验报告中简要说明你对proc_run函数的分析。

<del>如果下一个调度的进程不是当前进程，则为其分配CPU：
加锁，当前进程指针指向此进程，load_esp0加载此进程栈地址，lcr3加载PDT基地址，switch_to切换上下文。<del>

	1. 让current指向next内核线程initproc；
	2. 设置任务状态段ts中特权态0下的栈顶指针esp0为next内核线程initproc的内核栈的栈顶，
	即next->kstack + KSTACKSIZE ；
	3. 设置CR3寄存器的值为next内核线程initproc的页目录表起始地址next->cr3，这实际上是
	完成进程间的页表切换；
	4. 由switch_to函数完成具体的两个线程的执行现场切换，即切换各个寄存器，当switch_to
	函数执行完“ret”指令后，就切换到initproc执行了。

在本实验的执行过程中，创建且运行了几个内核线程？ 

	 创建且运行了idleproc、initproc两个线程

语句 local_intr_save(intr_flag);....local_intr_restore(intr_flag); 在这里有何作用?请说明理由  

	加锁？保证原子性？

注释：  
get_id,tf,context有待进一步研究其运作。
