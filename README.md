Here are some interview questions encountered during the job search process.

1. linux Index node 特性
文件是存储在硬盘上的，硬盘的最小存储单位叫做扇区 sector，每个扇区存储512字节。操作系统读取硬盘的时候，不会一个个扇区地读取，这样效率太低，而是一次性连续读取多个扇区，即一次性读取一个块block。这种由多个扇区组成的块，是文件存取的最小单位。块的大小，最常见的是4KB，即连续八个sector组成一个block。

文件数据存储在块中，那么还必须找到一个地方存储文件的元信息，比如文件的创建者、文件的创建日期、文件的大小等等。这种存储文件元信息的区域就叫做inode，中文译名为索引节点，也叫i节点。因此，一个文件必须占用一个inode，但至少占用一个block。


表面上，用户通过文件名打开文件，实际上，系统内部将这个过程分为三步：

1.系统找到这个文件名对应的inode号码；
2.通过inode号码，获取inode信息；
3.根据inode信息，找到文件数据所在的block，并读出数据。
其实系统还要根据inode信息，看用户是否具有访问的权限，有就指向对应的数据block，没有就返回权限拒绝。
使用 ls -i 直接查看文件i节点号，也可以通过stat查看文件inode信息查看i节点号。


2. memory_order_relaxed 标记的 store 操作，所以它们的执行顺序是不能保证的，也就是说可能被 CPU 或 编译器重排。
   一个对原子变量 A 的存储（store）操作如果施加了 memory_order_release 标记，它对内存操作的影响是：当前线程中这个存储操作之前的内存读、写操作都不可以被重排在这个存储操作之后
   其他线程中如果对原子变量 A 施加了带有 memory_order_acquire 标记的读取（load）操作，则当前线程中这个存储操作之前的其他内存读、写操作对这个线程可见
   其他线程中如果有对这个原子变量 A 施加了带有 memory_order_consume 标记的操作，则当前线程中所有原子变量 A 所依赖的内存写操作对其他线程可见（没有依赖关系的内存操作就不能保证顺序）
   这个例子再次说明了 C++ 的 std::memory_order 不是用来做线程同步的，它的意义仅仅在于当 consumer 线程中对原子变量 ptr 取到非 null 的值的时候，producer 线程中使用的 memory_order_release 标记能确保 ptr 在存储为非 null 值之前 #1 和 #2 的赋值操作已经完成（因为不会被重排到这个存储操作之后）。
    memory_order_acq_rel 简单理解，这个标记相当于对当前原子操作中的 load 操作施加了 memory_order_acquire 标记，对 store 操作施加了 memory_order_release 标记。于此同时，当前线程中这个操作之前的内存读写不能被重排到这个操作之后，这个操作之后的内存读写也不能被重排到这个操作之前。
3. 内存对齐可以提升读写效率
   分配对齐的内存
   #include <cstdlib> // 包含 posix_memalign 的声明 
void* aligned_memory = nullptr;  
const size_t alignment = 4096; // 4K 对齐  
const size_t size = ...; // 你需要的内存大小  
  
if (posix_memalign(&aligned_memory, alignment, size) == 0) {  
    // 内存分配成功，aligned_memory 指向对齐的内存块  
    // ... 使用内存 ...  
    free(aligned_memory); // 释放内存  
} else {  
    // 内存分配失败  
}

不对齐情况下占用的内存：全部长度加起来
对齐情况下：
对齐规则：
1.第一个成员在结构体变量偏移量为0 的地址处，也就是第一个成员必须从头开始。
2.其他成员的偏移量为对齐数**(该成员的大小 与 编译器默认的一个对齐数 中的较小值)**的整数倍。
3.结构体总大小对最大对齐数（通过最大成员来确定）的整数倍。
