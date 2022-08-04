操作系统






内核（kernel）：硬件与应用的中间人
  计算机是由各种外部硬件设备组成的，比如内存、cpu、硬盘等，如果每个应用都和这些硬件设备对接通信协议就太麻烦了，所以这个中间人就有内核来负责，让内核作为应用连接硬件设备的桥梁，应用程序只关心与内核交互，不用关心硬件细节。

	内核能力：
	  进程调度能力：管理进程、线程，决定哪个进程、线程使用CPU。
	  内存管理能力：管理内存，决定内存的分配与回收。
	  硬件通信能力：管理硬件设备，为进程与硬件设备之间提供通信能力。
	  系统调用能力：提供系统调用，应用程序要运行更高权限运行的服务，就需要系统调用，它是用户程序与操作系统之间的接口。
	


	内核工作：
	  内核权限很高，而应用程序的权限很小。一般把内存分为两个区域：
	    内核空间：只有内核程序可以访问。
		用户空间：专门给应用程序使用。
	  用户态：程序使用用户空间时。
	  内核态：程序使用内核空间时。
	  程序从用户态转变为内核态是通过系统调用完成的。

	



	Linux内核
	  MultiTask，多任务
	    多个任务同时执行，并发（单核）或并行（多核）
	  SMP，对称多处理
	    多个CPU地位相等，对资源使用权限相同，共享同一个内存，每个CPU都可以访问完整的内存和硬件资源。
		这决定了Linux OS不会有某个CPU单独服务应用程序或内核程序，每个程序都可以被分配到任意一个CPU上执行。
	  ELF，可执行文件连接格式
	  Monolithic Kernel，宏内核
	    Linux内核架构就是宏内核，意味着Linux的内核是一个完整的可执行程序，且拥有最高的权限。
		宏内核特征是系统内核的所有模块，比如进程调度、内存管理、文件系统、设备驱动等，都运行在内核态。
		微内核、混合类型内核





内存管理




进程管理





文件系统




设备管理




	
