# Plt 和Got

## plt 和 got 介绍

- PLT表，程序链接表 **Procedure Link Table**
- Got表，全局偏移表 **Global Offset Table**

PLT则是由代码片段组成的，每个代码片段都跳转到GOT表中的一个具体的函数调用

GOT是一个存储外部库函数的表



## 延迟绑定

在做`pwn`的时候 经常遇到 `plt`表和`got`表的联系，两者之间也

**只有动态库函数在被调用时，才会进行地址解析和重定位工作，这时候动态库函数的地址才会被写入到GOT表项中**

> 第一次调用动态库函数，调用过程如下图所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/5970003-bcf9343191848103.webp)

- 第一步由函数调用跳入到PLT表中，
- 第二步PLT表跳到GOT表中
- 第三步由GOT表回跳到PLT表中，这时候进行压栈，把代表函数的ID压栈
- 第四步跳转到公共的PLT表项中
- 第5步进入到GOT表中
- 然后_dl_runtime_resolve对动态函数进行地址解析和重定位
- 第七步把动态函数真实的地址写入到GOT表项中
- 然后执行函数并返回。

>解释下dynamic段，link_map和_dl_runtime_resolve
>
>dynamic段：提供动态链接的信息，例如动态链接中各个表的位置
>
>link_map：已加载库的链表，由动态库函数的地址构成的链表
>
>_dl_runtime_resolve：在第一次运行时进行地址解析和重定位工作



第二次调用过程，如下图：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/5970003-9baedd55881a39dd.webp)

可以看到：

- 第一步还是由函数调用跳入到PLT表
- 但是第二步跳入到GOT表中时，由于这个时候该表项已经是动态函数的真实地址了，所以可以直接执行然后返回。

**对于动态函数的调用，第一次要经过地址解析和回写到GOT表项中，第二次直接调用即可**



## 参考

[GOT表和PLT表 - 简书 (jianshu.com)](https://www.jianshu.com/p/0ac63c3744dd)

[【逆向学习记录】GOT表与PLT表_卦星的专栏-CSDN博客](https://blog.csdn.net/zhy025907/article/details/86088368)



