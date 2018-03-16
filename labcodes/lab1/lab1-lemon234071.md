本实验中重要知识点：  
*系统从加电启动到初始化gdt  
*中断异常  
*汇编  
*保护模式分段机制  
*函数堆栈  
*硬盘访问  
*ELF文件格式  
*调试UCORE  

很重要但未对应的知识点：  


练习一：  
1. 操作系统镜像文件ucore.img是如何一步一步生成的？(需要比较详细地解释Makefile中每一条相关命令和命令参数的含义，以及说明命令导致的结果)  

答：  
生成kern：  

	i386-elf-gcc -Ikern/xxx/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/xxx.c -o obj/kern/xxx.o  
	将kern/下.c文件生成.o文件  

	-Ixxx：指定目录  
	-fno-builtin：关闭内建库  
	-Wall：开启所有警告  
	-ggdb：让gcc为gdb生成比较丰富的调试信息  
	-m32：编译32位程序
	-gstabs：此选项以stabs格式生成调试信息 
	-nostdinc：不使用标准库 
	-fno-stack-protector：禁用堆栈粉碎保护器  
	
	i386-elf-ld -m elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/xxx.o  

	-m elf_i386：模拟为i386上的连接器  
	-nostdlib：不使用标准库  
	-T　xxx:让连接器使用指定的脚本  
	-o:将.o生成kernel文件  


生成bootloader:  

	i386-elf-gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o
	+ cc boot/bootmain.c
	
	i386-elf-gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o  
	
	-Os:为减小代码大小而进行优化  
	
	生成sign工具，生成bootblock
	gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o
	
	gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign  
	
	i386-elf-ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o  
	
	-N设置代码段和数据段都可读可写，关闭动态链接  
	-e start指定入口点符号为start  
	-Ttext 0x7C00设置代码段起始地址为0x7c00  
	
生成ucore.img

	dd if=/dev/zero of=bin/ucore.img count=10000  
	dd if=bin/bootblock of=bin/ucore.img conv=notrunc 
	dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
	
	if:输入文件
	of:输出文件
	count:复制块数
	/dev/zero 是一个字符设备，会不断返回0值字节（\0）
	conv=notrunc    输入文件的时候，源文件不会被截断
	seek=blocks     从输出文件开头跳过 blocks(512字节) 个块后再开始复制  

2.一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？  

		buf[512];  
		buf[510] = 0x55;  
		buf[511] = 0xAA;  

练习二：  
1.
1 修改 lab1/tools/gdbinit,内容为:  

	set architecture i8086  
	target remote :1234  
1.
2 在 lab1目录下，执行  

	make debug  

1.
3 在看到gdb的调试界面(gdb)后，执行如下命令，就可以看到BIOS在执行了  

	0x0000fff0
		(gdb)si
	0x0000e05b
		(gdb)si
1.		...
4 如果想看BIOS的代码  

	(gdb)x /2i 0xfffff0
		
	0xfffff0: add  %
	0xfffff2: add  %  

2.
执行

	(gdb)b *0x7c00  
	Breakpoint 1 at 0x7c00  
	(gdb)c
	(gdb)x/5i $pc
	0x7c00:		cli
	0x7c01:		cld
	0x7c02:		xor    %eax,%eax
    0x7c04:  	mov    %eax,%ds
	0x7c06:  	mov    %eax,%es
	(gdb)b 	
	
未完待续

练习三：  
为何开启A20，以及如何开启A20  
答：  

	在保护模式下，为了使能所有地址位的寻址能力，需要打开A20地址线控制，即需要通过向键盘控制器8042发送一个命令来完成。

如何初始化GDT表  
答：  
	
	lgdt gdtdesc
	以及对全局描述符表和任务状态段的初始化函数gdt_init

如何使能和进入保护模式  
答：  

	CR0寄存器使其中的保护模式使能位置位，ljmp $PROT_MODE_CSEG, $protcseg 进入了保护模式。

练习四：  
bootloader如何读取硬盘扇区的？  

	通过boot/bootmain.c中的readsect、readseg
	1. 等待磁盘准备好
	2. 发出读取扇区的命令
	3. 等待磁盘准备好
	4. 把磁盘扇区数据读到指定内存
	
bootloader是如何加载ELF格式的OS？  
elfhdr和bootmain注释很详细了

	/* bootmain - the entry of bootloader */
	void
	bootmain(void) {
    // read the 1st page off disk
    readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);

    // is this a valid ELF?
    if (ELFHDR->e_magic != ELF_MAGIC) {
        goto bad;
    }

    struct proghdr *ph, *eph;

    // load each program segment (ignores ph flags)
    ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
    eph = ph + ELFHDR->e_phnum;
    for (; ph < eph; ph ++) {
        readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
    }

    // call the entry point from the ELF header
    // note: does not return
    ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();

	bad:
    outw(0x8A00, 0x8A00);
    outw(0x8A00, 0x8E00);

    /* do nothing */
    while (1);
	}

补充：
Link addr& Load addr
Link Address是指编译器指定代码和数据所需要放置的内存地址，由链接器配置。LoadAddress是指程序被实际加载到内存的位置（由程序加载器ld配置）。一般由可执行文件结构信息和加载器可保证这两个地址相同。Link Addr和LoadAddr不同会导致：  
直接跳转位置错误  
直接内存访问(只读数据区或bss等直接地址访问)错误  
堆和栈等的使用不受影响，但是可能会覆盖程序、数据区域 注意：也存在Link地址和Load地址不一样的情况（例如：动态链接库）。  

练习五：

不一致如下：

	ebp:0xf000ff53 eip:0xf000ff53 args:0x0 0x0 0x0 0x0


