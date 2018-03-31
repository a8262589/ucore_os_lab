与参考答案对比：  
练习0：  

	1、proc结构体中未初始化proc->run_link，proc->lab6_run_pool 
	2、我trap.c中未删掉以前实验中的代码
练习二：  

	1、stride_enqueue、stride_pick_next中我未判断优先级为0时处理
  

本实验中重要知识点：  
  
	*调度框架  
	*stride算法  
	*rr算法
	*多级反馈队列调度算法
很重要但未对应的知识点：
	
	*实时调度和多处理器调度
	*优先级反置

练习1：

请理解并分析sched_calss中各个函数指针的用法，并接合Round Robin 调度算法描ucore的调度执行过程  

	// 初始化运行队列
	void (*init) (struct run_queue *rq);
	// 将进程 p 插入队列 rq
	void (*enqueue) (struct run_queue *rq, struct proc_struct *p);
	// 将进程 p 从队列 rq 中删除
	void (*dequeue) (struct run_queue *rq, struct proc_struct *p);
	// 返回 运行队列 中下一个可执行的进程
	struct proc_struct* (*pick_next) (struct run_queue *rq);
	// timetick 处理函数
	void (*proc_tick)(struct run_queue* rq, struct proc_struct* p);  
调度过程：
		
	当一个进程时间片用完，ucore将其插入就绪队列，并重置时间片，然后挑选下一个进程，然后将其从队列摘出，并运行。  

请在实验报告中简要说明如何设计实现”多级反馈队列调度算法“，给出概要设计，鼓励给出详细设计  

	维护多个队列，并判断条件从框架中调用不同队列参数。

练习2:   
实现 Stride Scheduling 调度算法（需要编码）
首先需要换掉RR调度器的实现，即用default_sched_stride_c覆盖default_sched.c。然后根据
此文件和后续文档对Stride度器的相关描述，完成Stride调度算法的实现。
后面的实验文档部分给出了Stride调度算法的大体描述。这里给出Stride调度算法的一些相关
的资料（目前网上中文的资料比较欠缺）。

	根据实验指导书和注释做即可。
	1、实验中stride_enqueue中时间片初始化有误，根据指导书改正。
	2、调用堆借口参数传递类型有误，修正。
	3、stride_pick_next中判空等号少一个，导致缺页异常。
	4、最大步长之差应在32位符号数表示范围之内。7FFFFFFF
	5、trap.c中发现lab1中计数用了while，应改为if，并修正此前的lab。
	6、schde.h中包含声明一下sched_class_proc_tick