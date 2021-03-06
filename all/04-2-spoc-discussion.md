#lec9: 虚存置换算法spoc练习

## 视频相关思考题

### 9.1 页面置换算法的概念

1. 设计置换算法时需要考虑哪些影响因素？如何评判的好坏？

需要考虑系统的行为模式，包括进程访问页面的规律；直观的评判标准是缺页率，同时可能也要考虑执行同样任务时页调入调出和替换的次数。

2. 全局和局部置换算法的不同？

全局置换算法考虑整个操作系统的内存管理，包含不同进程之间的调度；而局部性置换算法仅仅考虑单个进程（当前进程）的内存管理问题。

### 9.2 最优算法、先进先出算法和最近最久未使用算法

1. 最优算法、先进先出算法和LRU算法的思路？

最优算法：选择未来最久不会被访问的页面换出；但这个很难实现

先进先出算法：选择最先进入内存的页面换出。这实际上并不完全符合局部性原理，可能出现Belady现象。

LRU算法：找出最近最少使用的页面（最长时间未被访问的页面）换出，根据历史预测未来情况。

### 9.3 时钟置换算法和最不常用算法

1. 时钟置换算法的思路？

这可以认为是FIFO方法的一个简洁版实现。我们设定一个时钟位，一定程度上表示进入内存的顺序和被访问的情况；同时，将所有的页用循环链表存储。每次像时钟一样扫描所有页的时钟位；如果时钟位为0则将其换出，若为1则将其置零，并扫描下一个页，直到遇到时钟位为零的页面为止。

2. 改进的时钟置换算法与时钟置换算法有什么不同？

这里增加了修改位，可以判断页面被换出时是否需要被导出，或者跳过已经修改的页以增加换页的效率。

3. LFU算法的思路？

找出访问次数最少的页面将其换出。具体而言，是根据对页的访问次数进行排序。


### 9.4 Belady现象和局部置换算法比较

1. 什么是Belady现象？如何判断一种置换算法是否存在Belady现象？

Belady现象：当给进程分配的物理页面增加，缺页率反而上升。

可以构造反例说明其存在，也可以进行数学证明其不存在。

2. 请证明LRU算法不存在Belady现象。

简单来说，当给LRU算法分配更多的内存，这些内存一定包含在更少内存情况下的那些页面，因而它是严格更优的。

### 9.5 工作集置换算法

1. CPU利用率与并发进程数的关系是什么？

刚开始CPU利用率随并发进程数增加而增加，但达到一定程度后反而开始减少，这是因为在页面切换等开销上花费了过多时间。（局部性原理被违背）

2. 什么是工作集？

当前进程在使用的逻辑页面集合。其变化分为进程开始执行、工作集稳定和工作集过渡期。

3. 什么是常驻集？

当前时刻进程使用的实际驻留内存中的页面集合。

4. 工作集算法的思路？

并不仅仅因为缺页而将页换出，而是根据工作集的需求进行页面的换入和换出；目的是尽量让工作集留在内存中。一定时间未被访问的页面被视为离开工作集并自动换出内存，即使这时并没有待换出的页面。

### 9.6 缺页率置换算法

1. 缺页率算法的思路？

缺页率过低（小于某个值时），即缺页间隔较大时，控制给进程的内存分配，换入的同时也要换出；

反之，则缺页的时候仅仅换入，相当于增加内存。

### 9.7 抖动和负载控制

1. 什么是虚拟内存管理的抖动现象？

分配给进程的物理页面太少导致频繁置换，从而进程运行速度变慢。

2. 操作系统负载控制的最佳状态是什么状态？

各层内存均衡负载，并且互相不成为性能的瓶颈。平均缺页间隔 = 缺页异常处理时间。

3. 局部置换算法（如FIFO, LRU等）是否能作为全局置换算法来使用？为什么？

不能，因为进程切换会导致全部相关页面被换出，效率低下。

----

## 扩展思考题

1.  改进时钟置换算法的极端情况: 如果所有的页面都被修改过了，这时需要分配新的页面时，算法的performance会如何？能否改进在保证正确的前提下提高缺页中断的处理时间？

2.  如何设计改进时钟算法的写回策略?

3. （spoc）根据你的`学号 mod 4`的结果值，确定选择四种页面置换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试用例和测试结果。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)     

4. 请判断OPT、LRU、FIFO、Clock和LFU等各页面置换算法是否存在Belady现象？如果存在，给出实例；如果不存在，给出证明。

5. 了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！
	- 参考信息：
 	- [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
	 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
	 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
	 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
