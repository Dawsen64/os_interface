以请求页式分配为例。

1. 物理内存使用一个char数组进行模拟，定义了内存大小和页大小。
2. 操作系统启动时，需要新建一个内存对象，相当于启动内存， 

    ```MemoryManger mmu; ```
3. 需要分配内存时，使用分配函数，分配函数会返回一个类似寄存器的Page_table_regs*，表示该进程的所有内存信息，应放入PCB中。

    ```cpp
	/**
	 * @param vir_len 分配得虚拟页数， frame_len分配的物理帧数量
	 * @return 返回页表寄存器，应放入PCB中
	*/
	Page_table_regs* Alloc(int vir_len, int frame_len);
	
	//eg:
	mmu.Alloc(vir_len, frame_len);
	```

4. 在对内存进行读写操作或释放内存操作前，需要在进程切换时，调用Modify_page_table_register函数，函数的作用是把PCB中的Page_table_regs*传给内存对象，使内存处理对应进程。

	```cpp

	// 操作系统调用, 读写内存和释放内存时把PCB中的页表地址(序号)传入页表寄存器
	void Modify_page_table_register(Page_table_regs reg);
		
	//eg:
	mmu.Modify_page_table_register(*reg);
	mmu.Read(order * page_size, 1);
	mmu.Free();
	```
5. 进行读写时,向读写函数传入读写的首地址和待读的字节长度, 函数会返回一个对应长度的char数组.
	```cpp
	//读内存
	char* Read(int address, int len);
	//写内存
	int Write(int address, char* text, int len);

	//eg:
	int order = 1;
	//读写地址order*page_size开始的两个字节.
	mmu.Read(order * page_size, 2);
	mmu.write(order * page_size, 2);
	```

6. 释放内存内存时,调用Free函数即可.
	```cpp
	mmu.Free();
	```
测试用例: 
```cpp
//模拟两个进程，进程的访问队列相同，驻留集大小不同
//模拟访问队列,访问1,3,5,7...页
vector<int> access_array = { 1,3,5,7,9,3,2,4,6,8,10 };

MemoryManger mmu;
int page_size = mmu.Get_Page_SIZE();
//进程1虚拟地址64页, 4个驻留集
Page_table_regs* reg1 =  mmu.Alloc(64, 4);
//进程1虚拟地址64页, 6个驻留集
Page_table_regs* reg2 = mmu.Alloc(64, 6);
//切换到进程1
mmu.Modify_page_table_register(*reg1);
for (auto order : access_array) {
	mmu.Read(order * page_size, 1);
}
mmu.Free();
mmu.PrintStatus();
//切换到进程2
mmu.Modify_page_table_register(*reg2);
for (auto order : access_array) {
	mmu.Read(order * page_size, 1);
}
mmu.Free();
mmu.PrintStatus();
```


待解决的与文件管理接口的问题: 

假设一个进程的文件大小有10*1024字节, 即10K, 并且内存的页大小为1024字节,即1K. 当内存需要访问地址10*1024时,即进程文件的最后一个字节,按照操作系统课上的要求内存管理会把最后一页从文件系统中调入内存, 内存管理会提供的参数应该是文件名和页号第10页,如下 
```
char[] Call_into_memory(file_name, 10);
```
然后文件管理会把第10页以char[]返回.
写入同理.

