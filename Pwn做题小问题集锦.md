# 小问题集锦

## 栈题

### 栈对齐

特别注意到题目是部署在Ubuntu18上的，因此调用`system`需要栈对齐，这里填充ret来对齐；详细说明见如下链接

主要原因是[[原创\]CTF 100步getshell之就差一步——The MOVAPS issue-Pwn-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-269597.htm)

https://www.cnblogs.com/Rookle/p/12871878.html

简单总结：就是在64位的机器上，当你要调用printf或是system时，请保证`rsp&0xf==0`，说人话就是16字节对齐，最后4比特为0。当不满足上述条件的时候就报错。





## 堆题





## GDB使用

### 对开启PIE的程序调试

对于开启pie的程序，ida反汇编只会显示 相对偏移地址 (典型案例即为`libc`)

> 断点方式 为 `b *$rebse(0x1347)`

可能存在以下问题

> `ERROR: Could not find ELF base!`
>
> 主要原因是，使用`gdb` 的权限不够，请使用 `root` 用户 活着 `sudo` 提权使用