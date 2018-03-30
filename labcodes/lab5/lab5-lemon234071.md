与参考答案对比：  
练习0：  

	1、trap.c中我没有make sure当前current不为空，没有注释print_ticks 
	2、更新lab4初始化alloc_proc中索性全set 0
	3、更新lab4中fork时，没有仔细阅读set_links，没用正确使用，应更正。
	4、trap中，中断初始化与答案不一致（此处尚有异议）。
练习二：  

	1、copy_range中内存拷贝时，未能将线性地址转为void *导致失败,应更正。 
  

本实验中重要知识点：  
  
	*用户进程的创建  
	*子进程拷贝父进程资源  
	*进程状态（wait,exit等）  
	*进程管理调度  
	*系统调用
	*cow
	*用户进程代码的加载	
很重要但未对应的知识点：
	
	emmm

练习一：

请在实验报告中简要说明你的设计实现过程。  

	根据实验指导书，以及注释初始化即可。
请在实验报告中描述当创建一个用户态进程并加载了应用程序后，CPU是如何让这个应用程序最终在用户态执行起来的。即这个用户态进程被ucore选择占用CPU执行（RUNNING态）到具体执行应用程序第一条指令的整个经过。
 
	在本实验中第一个用户进程是由第二个内核线程initproc通过把hello应用程序执行码覆盖到initproc的用户虚拟内存空间来创建的。
	kernel_thread以init_main为参数调用do_fork创建用户进程。（此处没有找到init_main调用KERNEL_EXECVE(hello），但是看到创建新进程然后调用exit，经过调度，由父进程wait来回收）
	最终通过do_execve函数来完成用户进程的创建工作。
	load_icode加载应用程序执行码到当前进程的新创建的用户态虚拟空间中。
	用户环境已经搭建完毕。此时initproc将按产生系统调用的函数调用路径原路返回，执行中断返回指令“iret”（位于rapentry.S的最后一句）后，将切换到用户进程hello的第一条语句位置_start处（位于user/libs/initcode.S的第三句）开始执行。
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

