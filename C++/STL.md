# C++相关

## 1、C++的stl 

C++有六大部件：

容器（containers） 分配器（allocators）算法（algorithms） 迭代器（Iterators）适配器（adapters）仿函式（Functors）

[STL相关](https://blog.csdn.net/lxin_liu/article/details/89311749?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)

序列容器：vector   string   deque   list

关联容器：set   map   multiset   multimap

适配容器：stack   queue   priority_queue（优先队列）

## 1.1 vector

vector就是动态数组.

在堆中分配内存,元素连续存放,有保留内存,如果减少大小后，内存也不会释放.如果新值>当前大小时才会再分配内存.当该数组后的内存空间不够时，需要重新申请一块足够大的内存并进行内存的拷贝。内部使用allocator类进行内存管理，程序员不需要自己操作内存。对 vector 的任何操作，一旦引起空间重新配置，指向原 vector 的所有迭代器就都失效了。
常用函数
size()：当前vector元素个数
capacity()：vector已分配空间容量
reserve()：预分配空间,分配空间小于现有空间，不改变空间容量
resize()：改变size大小，不改变容量大小

### 1.2 list

底层为双向链表，内存空间不连续，只能通过指针访问数据，插入删除高效，随机存取非常没有效率。适用于对象需要大量删除插入操作的环境。
list的iterator不支持+，+=，<操作

### 1.3  Deque

Deque支持随机访问和存取，支持下标访问，插入删除的效率也很高。是由一段一段定量的连续空间组成的。

实现：map+buffer

插入元素时随时可以重新增加一段新的空间并连接起来。

![](https://img-blog.csdnimg.cn/20190415144854467.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4aW5fbGl1,size_16,color_FFFFFF,t_70)

提供的函数：

```
ush_back()  pop_back()  push_front()  pop_front()
Clear()  erase()  insert()
Deque.at()  deque.front()  deque.back()
Deque.begin()  deque.end()  deque.rbegin()  deque.rend()
```

Deque的迭代器:

![](https://img-blog.csdnimg.cn/20190415144920360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4aW5fbGl1,size_16,color_FFFFFF,t_70)

首先，它必须能够指出分段连续空间（亦即缓冲区）在哪里，其次它必须能够判断自己是否已经处于其所在缓冲区的边缘，如果是，一旦前进或后退就必须跳跃至下一个或上一个缓冲区。为了能够正确跳跃，deque必须随时掌握管控中心（map）。所以在迭代器中需要定义：当前元素的指针，当前元素所在缓冲区的起始指针，当前元素所在缓冲区的尾指针，指向map中指向所在缓区地址的指针，分别为cur, first, last, node。

### 1.4 Stack && queue

Stack和 queue是基于deque实现的 。stack和 queue没有迭代器，不提供走访功能。不使用vector作为底层的原因是vector扩容耗时。

### 1.5 heap

Heap不属于STL容器组件，它扮演priority queue的助手，作为其底层机制。Heap分为max_heap和min_heap两种，STL提供max_heap，底层以vector实现。

[关于堆的实现过程](https://blog.csdn.net/li1914309758/article/details/81036854)

初始化建堆过程时间：O(n)

插入 O(logn)

删除 O(logn)

### 1.6 priority_queue

带有权值观念，缺省情况下利用max_heap实现。不提供便利功能，也不提供迭代器。

```
priority_queue.top();
priority_queue.pop();
```

### 1.7 Map, Set, multiset, multimap 等实现原理

Map和set是基于红黑树实现的

Set中所有元素都会根据元素键值自动排序，不允许两个元素有相同键值，企图通过迭代器来改变set的元素是不被允许的。

### 1.8 Unordered_map

Unordered_map是哈希表，各项操作的平均时间复杂度接近常数。

可以自定义hash函数和比较函数

哈希函数：线性探测：H+i

​                  二次探测：H+i^2

​                  开链法

哈希函数用质数取模这就是为了使得有特征的数据集也能均匀映射到映射空间。 在这种情况下，有时hash函数还要结合数据特征，让出现概率较大的数据集有较小的碰撞概率，出现概率较小的数据集有较大的碰撞概率。这样，就可以减少整体数据集的碰撞概率。

### 1.9 allocators

构造和析构基本工具：construct()  destroy()

空间的配置和释放：std::alloc。SGI设计了双层级配置器，第一级配置器直接使用malloc()和free()，第二级配置器设计如下，采用不同的策略。

当请求的内存大于128b时，就用第一层配置器分配内存，当请求的内存小于等于128b时就调用第二层配置器。
这是由于使用malloc()的时候申请的内存会有一段固定的部分。所以当大于128b的时候，这部分就显得还可以接收，当小于128b的时候就会考虑使用不一样的策略。

![](https://note.youdao.com/yws/public/resource/3a0f04bd120995ed7cc98033b1d1b25a/xmlnote/WEBRESOURCE66233e5c891ce68845f1edeb3f3d7672/147)

第二层配置器维护16个自由链表（free lists），负责16种小型区块的次配置能力。有一个内存池，以malloc()配置而得。用一个union obj数组free_list来存储内存的地址，数组的每一个元素都指向一个obj链表，也就是内存链表。数组从小到大表示负责8b,16b,24b,...,120b,128b内存请求。当请求的内存为n时，会将请求上条至2的指数大小，并从数组相应位置获取内存。例如如果请求为20b，则请求会上调至24b 。
![](https://img-blog.csdnimg.cn/20190415145516941.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4aW5fbGl1,size_16,color_FFFFFF,t_70)

这个图展示了当有内存请求到达时，先找到负责这个内存大小的数组元素指向的内存链表，取出第一块内存，然后把数组元素(obj指针)指向第二块内存的首地址。

![](https://img-blog.csdnimg.cn/20190415145642431.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4aW5fbGl1,size_16,color_FFFFFF,t_70)

先计算机这块内存属于哪个数组元素负责，然后将这块回收的内存放置链表的第一个位置，这块内存的下一块内存为这个链表原先的第一块内存。

之前分析分配内存时，当free_list没有可用的内存时，会调用refill来从内存池分配内存。例如，如果请求内存为32b，此时内存链表中没有足够的内存了，那么refill会分配20块32b的内存块，然后把第一块返回给程序，其他19块由数组相应链表管理。
![](https://img-blog.csdnimg.cn/2019041514560634.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4aW5fbGl1,size_16,color_FFFFFF,t_70)

当free_list没有内存返回给用户时，refill函数会调用chunk_alloc从内存池获取内存，如果内存池剩余的内存(end_free-start_free)满足需求的内存(size*nobjs)，则直接从从内存池获取内存，返回给程序；当内存池的块不能满足20块时，返回一个以上的内存块；当内存池一块都不能满足时，先是回收剩余的内存，然后调用malloc从系统获取内存。如果系统内存不足，则先从数组其他链表获取内存，如果其他链表也不足够内存的话，就调用第一级内存来分配，因为第一级内存分配失败，有处理函数来解决。

## 2 堆内存和栈内存的区别

堆内存是区别于栈区、全局数据区和代码区的另一个[内存](https://baike.baidu.com/item/内存/103614)区域。堆允许程序在运行时动态地申请某个大小的内存空间。堆内存和栈内存的区别可以用如下的比喻来看出：使用堆内存就象是自己动手做喜欢吃的菜肴，比较麻烦，但是比较符合自己的口味，而且自由度大。使用栈内存就象我们去饭馆里吃饭，只管点菜（发出申请）、付钱和吃（使用），吃饱了就走，不必理会切菜、洗菜等准备工作和洗碗、刷锅等扫尾工作，他的好处是快捷，但是自由度小。

在标准[C语言](https://baike.baidu.com/item/C语言)上，使用[malloc](https://baike.baidu.com/item/malloc/659960)等内存分配函数获取内存即是从堆中分配内存，而在一个[函数体](https://baike.baidu.com/item/函数体)中例如定义一个[数组](https://baike.baidu.com/item/数组)之类的操作是从栈中分配内存。从堆中分配的内存需要程序员手动释放，如果不释放，而系统[内存管理器](https://baike.baidu.com/item/内存管理器)又不自动回收这些堆内存的话，那就一直被占用。我们掌握堆内存的权柄就是返回的[指针](https://baike.baidu.com/item/指针)，一旦丢掉了指针，便无法在我们视野内释放它。这便是[内存泄露](https://baike.baidu.com/item/内存泄露)。

