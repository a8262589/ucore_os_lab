与参考答案对比：  
练习0：  

	1、trap.c中我没有make sure当前current不为空，没有注释print_ticks 
	2、更新lab4初始化alloc_proc中索性全set 0
	3、更新lab4中fork时，没有仔细阅读set_links，没用正确使用，应更正。
	4、trap中，中断初始化与答案不一致（此处尚有异议）。
练习二：  

	1、copy_range中内存拷贝时，未能将线性地址转为void *导致失败,应更正。 
  

本实验中重要知识点：  
  
	*进程切换、调度  
	*RR调度算法  
	*进程状态
很重要但未对应的知识点：
	
	*其他调度算法
	*优先级反置

练习一：

请理解并分析sched_calss中各个函数指针的用法，并接合Round Robin 调度算法描ucore的调度执行过程  

	
请在实验报告中简要说明如何设计实现”多级反馈队列调度算法“，给出概要设计，鼓励给出详细设计  


练习二：

请在实验报告中简要说明你的设计实现过程。  

	根据注释和指导书完成代码，make qemu发生缺页异常，没有将线性地址转为void*进行memset操作。此处不知为什么，代码中其他处直接用page2kva作为参数并未出错。
请在实验报告中简要说明如何设计实现”Copy on Write 机制“，给出概要设计，鼓励给出详细设计。
	
	emmm...。
练习三： 

请在实验报告中简要说明你对 fork/exec/wait/exit函数的分析。  

	
并回答如下问题：  
请分析fork/exec/wait/exit在实现中是如何影响进程的执行状态的？  

	do_fork调用alloc_proc创建子进程，此时子进程状态为PROC_UNINIT，后通过唤醒子进程使其PROC_RUNNABLE，
	exec来加载应用程序代码运行.
	exit释放进程自身所占内存空间和相关内存管理（如页表等）信息所占空间此时子进程状态PROC_ZOMBIE，唤醒父进程。
	do_wait父进程等待子进程此时可能使父进程PROC_SLEEPING，并在得到子进程的退出消息后，彻底回收子进程所占的资源。
请给出ucore中一个用户态进程的执行状态生命周期图（包执行状态，执行状态之间的变换关系，以及产生变换的事件或函数调用）。（字符方式画即可）  

	  alloc_proc                                 RUNNING
	      +                                   +--<----<--+

	      +                                   + proc_run +

	      V                                   +-->---->--+ 
	PROC_UNINIT -- proc_init/wakeup_proc --> PROC_RUNNABLE -- try_free_pages/do_wait/do_sleep --> PROC_SLEEPING --
	                                           A      +                                                           +
	                                           |      +--- do_exit --> PROC_ZOMBIE                                +
	                                           +                                                                  + 
	                                           -----------------------wakeup_proc----------------------------------

