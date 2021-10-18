# Ret2reg

---

`Ret2reg`整体上与`Ret2Shellcode`原理上基本一致，只是在返回地址的控制有所区别；在Ret2shellcode中，通过栈溢出，构造目标调用地址将栈中的`ret`地址覆盖，从而使函数进行跳转至我们想要的地址(即shellcode写入的地址，也就buf地址，该地址若未开启ASLR，则固定不变，在程序运行前即可反汇编来确定。)，而`Ret2reg`针对的是开启ASLR保护的程序，借用寄存器指向buf，通过调用寄存器地址来进行跳转。**此类漏洞常见于strcpy字符串拷贝函数中。**

+++

### 参考文档

[中级ROP之ret2reg_至臻求学，胸怀云月-CSDN博客](https://blog.csdn.net/AcSuccess/article/details/104465113)

### Target

该技术主要是为了绕过ASLR技术的保护！！!



### 原理

- 查看栈溢出返回时哪个寄存器指向缓冲区空间。
- 查找对应的call 寄存器或者jmp 寄存器指令，将EIP设置为该指令地址。
- 将寄存器所指向的空间上注入shellcode（确保该空间是可以执行的，通常是栈上的）

### 利用思路

- 分析和调试汇编，查看溢出函数返回时哪个寄存器指向缓冲区地址
- 向寄存器指向的缓冲区中注入shellcode
- 查找call 该寄存器或者jmp 该寄存器指令，并将该指令地址覆盖ret



### 防御方法

在函数ret之前，将所有赋过值的寄存器全部复位，清0，以避免此类漏洞



### 实验

#### ASLR

ASLR （Address Space layout randomization）有三种状态，关闭，部分开启，完全开启；该题需要ALSR完全开启；相关指令如下

```python
cat  /proc/sys/kernel/randomize_va_space   #查看ASLR的参数值
0  -  No  randomization
1  -  Conservative  randomization
2  -  Full  randomization.

若ASLR的参数值非2，可通过以下指令进行设置
echo 2 > /proc/sys/kernel/randomize_va_space   #将参数值设置为2
```

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-04-30_16-50-39.png)



#### 源代码

```C
#include <stdio.h>
#include <string.h>
void evilfunction(char *input) {
        char buffer[512];
        strcpy(buffer, input);
}
int main(int argc, char **argv) {
        evilfunction(argv[1]);
        return 0;
}

```



#### 编译

编译指令如下：

>gcc -Wall -g -o reg reg.c -z execstack -m32 -fno-stack-protector





#### 断点

首先是查看函数，使用指令

> info function

然后下断点追踪，由于开启了ASLR，每次运行程序，加载的地址都会有所不同，因此在运行程序之前通过`disass function`的方式正对固定地址下断点是无效的。

因此我们需要 正对函数名下断点

> gdb --args rge 123456 #使用123456作为参数调试reg程序
>
> b evilfunction  对该函数下断点
>
> 在上诉函数断下来以后  在使用disass function 然后正对ret的具体具体地址进行断点

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-05-06_20-02-01.png)

如上图所示，展示了 `evilfunction`函数的反汇编，并指出了断点位置



#### 寄存器

由下图可以看到，当在`evilfunction`函数中运行时，eax，ecx，esp都指向了输入的字符串“123456”，此时`evilfunction`函数即将返回到`main`函数

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-05-06_20-04-46.png)

进一步，向下跟进，可以得到下图：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-05-06_20-11-57.png)

由上图可知，当程序已退出`evilfunction`函数，运行在`main`函数时，寄存器eax，ecx，edx没有处理，依旧保存着原始的值，指向输入的字符串“123456”；这就可以被我们所运运用



#### ROPgadget

> objdump -d ret2reg | grep *%eax
>
> ROPgadget --binary  reg --only 'call|ret'|grep eax  #功能和上面一样



### Payload

```python
payload = shellcode + (offset + 4 - len(shellcode)) * a + p32(call_eax_addr)
```



