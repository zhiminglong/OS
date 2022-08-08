# 操作系统

参考 [coding](https://xiaolincoding.com/os/)

## 内核（kernel）：硬件与应用的中间人

计算机是由各种外部硬件设备组成的，比如内存、cpu、硬盘等，如果每个应用都和这些硬件设备对接通信协议就太麻烦了，所以这个中间人就有内核来负责，让内核作为应用连接硬件设备的桥梁，应用程序只关心与内核交互，不用关心硬件细节。

---

- 内核能力：
  - 进程调度能力：管理进程、线程，决定哪个进程、线程使用CPU。
  - 内存管理能力：管理内存，决定内存的分配与回收。
  - 硬件通信能力：管理硬件设备，为进程与硬件设备之间提供通信能力。
  - 系统调用能力：提供系统调用，应用程序要运行更高权限运行的服务，就需要系统调用，它是用户程序与操作系统之间的接口。
	
---

- 内核工作：
  - 内核权限很高，而应用程序的权限很小。一般把内存分为两个区域：
    - 内核空间：只有内核程序可以访问。
    - 用户空间：专门给应用程序使用。
  - 用户态：程序使用用户空间时。
  - 内核态：程序使用内核空间时。
  - 程序从用户态转变为内核态是通过系统调用完成的。

---	

- Linux内核
  - MultiTask，多任务
	- 多个任务同时执行，并发（单核）或并行（多核）
  - SMP，对称多处理
	- 多个CPU地位相等，对资源使用权限相同，共享同一个内存，每个CPU都可以访问完整的内存和硬件资源。
	- 这决定了Linux OS不会有某个CPU单独服务应用程序或内核程序，每个程序都可以被分配到任意一个CPU上执行。
  - ELF，可执行文件连接格式
  - Monolithic Kernel，宏内核
	- Linux内核架构就是宏内核，意味着Linux的内核是一个完整的可执行程序，且拥有最高的权限。
	- 宏内核特征是系统内核的所有模块，比如进程调度、内存管理、文件系统、设备驱动等，都运行在内核态。
	- 微内核、混合类型内核
---

> ## 一、内存管理

### 虚拟内存：
- 不同进程的虚拟地址和不同内存的物理地址映射起来，该机制由操作系统提供。
   - 虚拟内存地址：用户程序所使用的内存地址。
   - 物理内存地址：实际存在硬件里面的空间地址。
- MMU：内存管理单元（*Memory Management Unit*），管理虚拟内存与物理地址的映射关系。
- 虚拟内存作用
	- 虚拟内存可以使得进程对运行内存超过物理内存大小，因为程序运行符合局部性原理，CPU访问内存会有很明显的重复访问的倾向性，对于那些没有被经常使用到的内存，我们可以把它换出到物理内存之外，比如硬盘上的swap区域。
	- 每个进程都有自己的页表，所以每个进程的虚拟内存空间就是相互独立的。进程也没有办法访问其他进程的页表，所以这些页表是私有的，这就解决了多进程之间地址冲突的问题。
	- 页表里的页表项中除了物理地址外，还有一些标记属性的比特，比如控制一个页的读写权限，标记该页是否存在等。在内存访问方面，操作系统提供了更好的安全性。
---

#### 内存分段

程序是由若干逻辑分段组成，如代码分段、数据分段、栈段、堆段。不同的段是有不同的属性的，所以就用**分段**（*Segmentation*）的形式把这些段分离出来。

   虚拟地址与物理地址之间通过**段表**来映射。

- 虚拟地址组成：段选择因子和段内偏移量
  - 段选择因子保存在段寄存器里，因子中的段号被用作段表的索引。段表里面保存的是段的基地址、段的界限和特权等级等。
  - 段内偏移量位于0和段界限之间，段基址加上段内偏移量得到物理内存地址。

- 分段优点：产生连续的内存空间

- 分段缺点：内存碎片、内存交换效率低
  - 内存碎片：
	- 外部内存碎片：产生了多个不连续的小物理内存，导致新的程序无法被装载。
	  - 【解决问题】 内存交换（*Swap*）

		内存交换：将已装载的程序暂时写到硬盘，然后再读回内存，但是装载回来的位置改变，将内存中的不连续的内存碎片组合成连续的，便又能装载其他程序。

	- 内部内存碎片：程序所有的内存都被装载到了物理内存，但是程序有部分内存并不常用，导致内存浪费。
	  - 【解决问题】 内存分页

  - 内存交换效率低：
	- 对于多进程的系统来说，采用分段方式，内存碎片很容易产生，导致经常进行内存交换，但是硬盘的访问速度比内存慢太多，特别是交换大内存空间的程序，会显得机器卡顿。
	- 【解决问题】 内存分页
---

#### 内存分页
分页就是把整个虚拟和物理内存空间切成一段段固定尺寸的大小。这样一个连续并且尺寸固定的内存空间，叫做**页**（*Page*）。Linux下，每一页的大小为`4KB`。

   虚拟地址与物理地址之间通过**页表**来映射。

- 缺页异常
	- 当进程访问的虚拟地址再也表中查不到时，系统便会产生一个**缺页异常**，进入系统内核空间分配物理内存、更新进程页表，最后再返回用户空间，恢复进程的运行。

- 分页优点
	- 无内存碎片：
		- 内存空间都是预先划分好的，就不会像分段产生间隙非常小的内存。
		- 释放的内存都是以页为单位释放的，也就不会产生无法给进程使用的小内存。
	- 内存交换效率较高：
		- 内存空间不够时，OS会把正在运行的进程的最近没被使用的内存页面释放掉，暂时写在硬盘上，成为**换出**（*Swap Out*）。一旦需要的时候再加载进来，称为**换入**（*Swap In*）。一次性写入硬盘的只有少数的一个页或者几页，需要的时间降低。

- 虚拟地址由**页号**和**页内偏移**组成
	- 页号：页表的索引，**页表**包含物理页每页所在物理内存的基地址，基地址与页内偏移就形成了物理内存地址。
	- 页内偏移：实际物理地址相对于该页物理地址的基地址的偏移量。

- 单分页缺点：占内存

- **多级页表**（*Multi-Level Page Table*）

  在多级页表中，一级页表覆盖整个虚拟地址空间，如果某个一级页表的页表项没有被用到，也就不需要创建这个页表项对应的二级页表了，及可以**在需要时才创建二级页表**。在4GB的虚拟地址空间中（32位系统，每个页表项占4个字节），采用二级分页，一级页表有1024个页表项（二级页表），每个二级页表包含1024个页表项，假设只有20%的一级页表项被用到，那么页表占用的空间为：`4KB（一级页表）+20% * 4MB（二级页表）= 0.804MB`，相较于单级页表`4MB`节约了大量内存。 **页表一定要覆盖全部虚拟地址空间，不分级的页表需要100多万个页表项来映射，而二级分页只需要1024个页表项。**
	- 64位系统中，采用了四级目录：
		- 全局页目录项PGD(*Page Global Directory*)
		- 上层页目录项PUD(*Page Upper Directory*)
		- 中间页目录项PMD(*Page Middle Directory*)
		- 页表项 PTE（*Page Table Entry*）

	- TLB
	
	多级页表虽然解决了空间上的问题，但是虚拟地址到物理地址转换工序增加，降低地址转换速度，带来了时间上的开销。
	
	程序是有局限性的，即在一段时间内，整个程序的执行仅限于程序中的某一部分。相应的，执行所访问的存储空间也局限于某个内存区域。利用这个特性，把最常访问的几个页表项存储到访问速度更快的硬件，即CPU中加入一个专门存放程序最常访问的页表项的Cache，这个Cache就是TLB（*Translation Lookaside Buffer*），通常称为页表缓存、转址旁路缓存、快表等。

---

#### 段页式内存管理
 
   内存分段和内存分页并不是对立的，它们可以组合起来在同一个系统中使用，称为**段页式内存管理**。
- 实现方式
	1. 先将程序划分为多个有逻辑意义的段，也就是分段机制；
	2. 接着把每个段划分为多个页，也就是对分段划分出来的连续空间，再划分固定大小的页。

- 虚拟地址结构由**段号、段内页号和页内位移**三部分组成。

- 段页式地址变换重要得到物理地址须经过三次内存访问
	- 第一次访问段表，得到页表起始地址；
	- 第二次访问页表，得到物理号；
	- 第三次将物理页与页内位移组合，得到物理地址。

---

### Linux内存管理

---

### malloc分配内存

在用户空间内存中存在以下6中内存段（从低到高）：
- 程序文件段，包括二进制可执行代码；
- 已初始化数据段，包括静态常量；
- 未初始化数据段，包括未初始化的静态变量；
- 堆段，包括**动态分配**的内存，从低地址开始向上增长；
- 文件映射段，包括动态库、共享内存等，从低地址开始向上增长；
- 栈段，包括局部变量和函数调用的上下文等。栈的大小是固定的，一般是`8MB`，支持自定义。

  `malloc()`或者`mmap()`分别在堆和文件映射段动态分配内存。
 - malloc
	- 通过brk()系统调用从堆分配内存
	   - 通过brk()函数将堆顶指针向高地址移动，获得新的内存空间
	   - free释放内存的时候，并不会把内存归还给操作系统，而是缓存在malloc的内存池中，待下次使用。

	- 通过mmap()系统调用在文件映射区域分配内存
	   - mmap()系统调用私有匿名映射的方式，在文件映射区分配一块内存，也就是从文件映射区偷一块内存。
	   - free时，会把内存归还给操作系统，内存得到真正的释放。

**mallos()分配的是虚拟内存。**

---

### 内存满了怎么办：内存回收

**后台内存回收**（kswapd）：物理内存紧张时，唤醒kswapd内核线程来回收内存，这个回收内存的过程**异步**的，不会阻塞进程的执行。


**直接内存回收**（direct reclaim）：如果后台异步回收跟不上进程内存申请的速度，就会开始直接回收，这个回收内存的过程是**同步**，会阻塞进程的执行。

***OOM（Out of Memory）机制***
  
   当直接内存回收后，空闲的物理内存仍然无法满足此次物理内存的申请，便会触发OOM机制。

   OOM Killer机制会根据算法选择一个占用物理内存比较高的进程，将其杀死，释放内存资源，如果物理内存依然不足，会继续杀死占用内存较高的进程，直到释放足够的内存位置。可以通过调整进程的/proc/[pid]/oom_score_adj值来降低被OOM killer杀掉的概率。

- 被回收内存类型
 - **文件页**（*File-backed Page*）
	- 内核缓存的磁盘数据（Buffer）和内核缓存的文件数据（Cache）都叫作文件页。
	- 对于干净页是直接内存释放，不会影响性能；对于脏页会先写回到磁盘再释放内存，这个操作会发生磁盘I/O的，会影响系统性能。

 - **匿名页**（*Anonymous Page*）
	- 这部分内存没有实际载体，不像文件缓存有硬盘文件这样一个载体，比如堆、栈数据等。
	- 它回收的方式是通过**Linux的Swap机制**，Swap会把不常访问的内存先写到磁盘中，然后释放这些内存，给其他更需要的进程使用。再次访问这些内存时，重新从磁盘读入内存就可以了。

文件页和匿名页的回收都是基于LRU算法，也就是先回收不常访问的内存。回收内存的操作基本上都会发生磁盘I/O，如果回收内存操作频繁，意味着磁盘I/O次数会很多，势必会影响系统性能。

---

### Swap机制

Swap就是把一块磁盘空间后者本地文件，当成内存来使用，它包含换出和换入两个过程：
- **换出**（*Swap Out*），是把进程暂时不用的内存数据存储到硬盘中，并释放这些数据占的内存；
- **换入**（*Swap In*），是进程再次访问这些内存的时候，把它们从磁盘读到内存中来。

触发场景：
- **内存不足**：当系统需要的内存超过了可用的物理内存时，内核会将内存中不常用的内存页交换到磁盘上为当前进程让出内存，保证正在执行的进程的可用性，这个内存回收的过程时强制的直接内存回收。
- **内存闲置**：应用程序在启动阶段使用的大量内存在启动后往往都不会使用，通过后台运行的守护进程（kswapd），我们可以将这部分只使用一次的内存交换到磁盘上为其他内存的申请预留空间。

优点：
- 应用程序实际可以使用的内存空间将远远超过系统的物理内存。

缺点：
- 频繁地读写硬盘，会显著降低操作系统的运行速率。


>## 二、进程管理

- diyi
- dier
  - sjkd
- disan





>## 三、文件系统




>## 四、设备管理




	
