---
title: 2024A-stage4-arceos-unikernel1-Martin1847
date: 2024-12-20 19:38:11
tags:
    - author:martin1847
    - repo:https://github.com/martin1847/arceos
---

# Arceos 训练营总结

转眼间两个多月的训练营要进入尾声了。
还记得国庆回山东的高铁上，在刷`rustlings`的算法题，小根堆的实现。

从二阶段的`rCore`到组件化的`arceos`,一路下来，对OS几大部件`CPU`虚拟化、`内存`虚拟化和文件系统虚拟化有了更深刻的理解。

下面我也从这几个方面来分别做个总结。




<!-- more -->

## CPU虚拟化

了解了`CPU`的几种特权模式，以及到OS进行系统调用进行的硬件级别（汇编指令）支持。

包括通过对时钟中断的支持，完成抢占式调度（这也解释了在用户态较难实现lock）。

在CPU眼里，没有函数、没有内存分配，有的只是指令和数据。

对于高级语言而言，一个用户态程序的最终运行，是`编译器`和`os`和CPU共同协同运作的结果。

编译器负责栈的初始化，函数上下文的处理，os负责提供硬件资源的抽象、系统调用，CPU提供特权指令。

虚拟化的目的，是为了多个程序的调度，让每个程序“独占”CPU，减少了编码的心智负担。

## 内存虚拟化

这里印象深刻的是分页，页表实现。硬件层面的`MMU`和`TLB`支持。

我们实现了彻底分页，内核态和用户态是两个页表。通过分页保障了安全，也方便了应用编写。

这一块也是非常复杂的点。

印象比较深刻的是三级页表的内存占用，以及`mmap`的实现。通过`mmap`可以从内核申请共享内存、或者映射文件，减少系统调用次数。这里也是默认`lazy init`,用到的时候通过缺页中断去真正完成映射。

通过分页`PTE`也很好了保护了内存安全，是否可执行。

![riscv-sv39-pte](https://rcore-os.cn/rCore-Tutorial-Book-v3/_images/sv39-pte.png)

另外一点就是内存分配算法，内存本质就是一个大的`byte[]`,找到一个可用的（按页不按页都可以）地址，也即`index`返回即可。

开启分页后`MMU`会进行更多安全检查。比如“数据执行保护”（Data Execution Prevention, DEP）或“不可执行位”（No-eXecute, NX），CPU碰到不可执行的代码区，会触发异常。起到防注入、溢出等的保护。



## 文件系统虚拟化

了解了ELF文件格式，静态link，动态link，以及文件inode的底层细节。

设计一个高效的文件系统是个复杂的工程，本质根内存分配一样，要减少内部碎片、外部碎片，而磁盘访问速度远远不及内存，真是一件复杂的事情。

# 总结

计算机没有魔法。通过这阵子的学习，对于CPU、内存如何协同工作加深了理解。
哪些是硬件的支持，哪些是OS的抽象、编译器的辅助，有了更深刻的认识。
比如栈的处理，编译器处理函数上下文、OS初始化栈、CPU提供栈指针寄存器，共同协作完成。
这是一次很好的练习，对于从最底层看本质非常有帮助。

感谢清华大学的主办，感谢各阶段的老师们的指导！
