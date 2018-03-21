本实验中重要知识点：  
*虚拟内存管理  
*页面换入换出  
*完善页表项映射  
*页访问异常处理  
*页替换算法  
*cache局部性原理（在数据结构mm_struct中）

很重要但未对应的知识点：


练习一：

1、了解do_pagefault调用关系、虚地址空间和物理地址空间关系图
  
2、为了完成 do_pagefault了解数据结构以及函数  
vmm\_struct：//ucore中描述应用程序对虚拟内存“需求”

	struct vma_struct {
	// the set of vma using the same PDT
	struct mm_struct *vm_mm;
	uintptr_t vm_start; // start addr of vma
	uintptr_t vm_end; // end addr of vma
	uint32_t vm_flags; // flags of vma
	//linear list link which sorted by start addr of vma
	list_entry_t list_link;
	};  

	list_link是一个双向链表，按照从小到大的顺序把一系列用vma_struct表示的虚拟内存空间链接起来，并且还要求这些链起来的vma_struct应该是不相交的  
	vm_flags表示了这个虚拟内存空间的属性  
mm_struct：  

	struct mm_struct {
	// linear list link which sorted by start addr of vma
	list_entry_t mmap_list;
	// current accessed vma, used for speed purpose
	struct vma_struct *mmap_cache;
	pde_t *pgdir; // the PDT of these vma
	int map_count; // the count of these vma
	void *sm_priv; // the private data for swap manager
	};

	mmap_list是双向链表头，链接了所有属于同一页目录表的虚拟内存空间  
	mmap_cache是指向当前正在使用的虚拟内存空间，“局部性”原理	 
	pgdir 所指向的就是 mm_struct数据结构所维护的页表  
	map_count记录mmap_list里面链接的 vma_struct的个数。  
	sm_priv指向用来链接记录页访问情况的链表头，这建立了mm_struct和swap_manager之间的联系。

相关函数：  

	find_vma根据输入参数addr和mm变量，查找在mm变量中的mmap_list双向链表中某个vma包含此addr，即vma->vm_start<=addr end。

3、了解swap_in函数：  
分配一个物理页，根据PTE内容找到磁盘区地址，从磁盘读入内容到物理页上。  

4、重新详细了解Page Fault异常处理  
当启动分页机制以后，如果一条指令或数据的虚拟地址所对应的物理页框不在内
存中或者访问的类型有错误（比如写一个只读页或用户态程序访问内核态的数据等），就会发生页访问异常。即无法完成从虚拟地址到物理地址映射时，执行中断服务例程实现“按需调页”/“页换入换出”。  

产生页访问异常后，CPU把引起页访问异常的线性地址装到寄存器CR2中，并给出了出错码errorCode。do_pgfault函数是完成页访问异常处理的主要函数，它根据从CPU的控制寄存器CR2
中获取的页访问异常的物理地址以及根据errorCode的错误类型来查找此地址是否在某个VMA的地址范围内以及是否满足正确的读写权限，如果在此范围内并且权限也正确，这认为这是一次合法访问，但没有建立虚实对应关系。所以需要分配一个空闲的内存页，并修改页表完成虚地址到物理地址的映射，刷新TLB，然后调用iret中断，返回到产生页访问异常的指令处重新执行此指令。如果该虚地址不在某VMA范围内，则认为是一次非法访问。  

5、理解原理和do_pagefault要求实现的功能后按注释填入代码。每步代码要理解原理，使用方式。新页结构虚拟地址需要赋予addr。 与答案对比，缺少操作失败的判断。 

*请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中组成部分对ucore实现页替换算法的潜在用处。  
描述PDE：  

 	31                12 11 10 9  8 7 6 5 4 3 2 1 0  
	+--------------------+-------+-+-+-+-+-+-+-+-+-+  
	|                    |       | |P| | |P|P|U|R| |  
	|   页表基址高20位     |忽略   |G|S|0|A|C|W|/|/|P|  
	|                    |       | | | | |D|T|S|W| |   
	+--------------------+-------+-+-+-+-+-+-+-+-+-+ 
描述PTE：  

 	31                12 11 10 9  8 7 6 5 4 3 2 1 0  
	+--------------------+-------+-+-+-+-+-+-+-+-+-+  
	|                    |       | | | | |P|P|U|R| |  
	|   页基址高20位       |忽略   |G|0|D|A|C|W|/|/|P|  
	|                    |       | | | | |D|T|S|W| |  
	+--------------------+-------+-+-+-+-+-+-+-+-+-+  

【A】：访问位。clock算法计数。  
【D】：脏位。改进clock算法记录修改。   
【P】：存在位。若为0，接下来的7位暂时保留，可以用作各种扩展；而原来用来表示页帧号的高24位地址。  

*如果ucore的缺页服务例程在执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？  


练习二：  
页置换机制：  
如果一个页（4KB/页）被置换到了硬盘某8个扇区（0.5KB/扇区），该PTE的最低位--present位应该为0 （即 PTE_P 标记为空，表示虚实地址映射关系不存在），接下来的7位暂时保留，可以用作各种扩展；而原来用来表示页帧号的高24位地址，恰好可以用来表示此页在硬盘上的起始扇区的位置（其从第几个扇区开始）。  

1、按照注释完成代码，此处注意page结构有所变化，le2page参数要变。

2、


