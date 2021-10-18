# GDB 调试技巧总结

---

主要是在 Pwn 学习中用到的调试技巧和指令加以总结

[GDB的那些奇淫技巧](https://evilpan.com/2020/09/13/gdb-tips/#%E8%BF%90%E8%A1%8C%E7%A8%8B%E5%BA%8F)

[100个gdb小技巧](https://wizardforcel.gitbooks.io/100-gdb-tips/content/show-version.html)

---

- disass -filename

  反汇编某一函数 得到该函数的汇编代码 乐意与 objdump 一起使用(objdump可以得到所拥有的函数及其函数名)

### 功能分类

#### 1、运行命令

run：（简写 r） ，其作用是运行程序，当遇到断点后，程序会在断点处停止运行，等待用户输入下一步的命令。
continue （简写c ）：继续执行，到下一个断点处（或运行结束）
next：（简写 n），单步跟踪程序，当遇到函数调用时，也不进入此函数体；此命令同 step 的主要区别是，step 遇到用户自定义的函数，将步进到函数中去运行，而 next 则直接调用函数，不会进入到函数体内。
step （简写 s）：单步调试如果有函数调用，则进入函数；与命令n不同，n是不进入调用的函数的
until：（简写 u）当你厌倦了在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体。
until+行号： 运行至某行，不仅仅用来跳出循环
finish： 运行程序，直到当前函数完成返回，并打印函数返回时的堆栈地址和返回值及参数值等信息。
call 函数(参数)：调用程序中可见的函数，并传递“参数”，如：call gdb_test(55)
quit：（简写 q） ，退出gdb

#### 2、设置断点 

break n （简写b n）:在第n行处设置断点
（可以带上代码路径和代码名称： b OAGUPDATE.cpp:578）
b fn1 if a＞b：条件断点设置
break func（break简写 b）：在函数func()的入口处设置断点，如：break cb_button
delete （简写 d）断点号n：删除第n个断点
disable 断点号n：暂停第n个断点
enable 断点号n：开启第n个断点
clear 行号n：清除第n行的断点
info break （info breakpoints）（简写 i b） ：显示当前程序的断点设置情况
delete breakpoints：（简写 de break）清除所有断点：

#### 3、查看源码

list ：（简写 l） ，其作用就是列出程序的源代码，默认每次显示10行。
list 行号：将显示当前文件以“行号”为中心的前后10行代码，如：list 12
list 函数名：将显示“函数名”所在函数的源代码，如：list main
list ：不带参数，将接着上一次 list 命令的，输出下边的内容。

#### 4、显示寄存器

info reg （简写 i r）可以显示寄存器内容

在寄存器名之前加$，可以显示寄存器内容。
p $寄存器：显示寄存器内容
p/x $寄存器：十六进制显示寄存器内容。

用x命令可以显示内容内容，“x/格式 地址”。
x $pc：显示程序指针内容
x/i $pc：显示程序指针汇编。
x/10i $pc：显示程序指针之后10条指令。
x/128wx 0xfc207000：从0xfc20700开始以16进制打印128个word。

还可以通过disassemble指令来反汇编。
disassemble
disassemble 程序计数器 ：反汇编pc所在函数的整个函数。
disassemble addr-0×40,addr+0×40：反汇编addr前后0×40大小。

#### 5、打印表达式

print 表达式：（简 p） ，其中“表达式”可以是任何当前正在被测试程序的有效表达式，比如当前正在调试C语言的程序，那么“表达式”可以是任何C语言的有效表达式，包括数字，变量甚至是函数调用。
print a：将显示整数 a 的值
print ++a：将把 a 中的值加1,并显示出来
print name：将显示字符串 name 的值
print gdb_test(22)：将以整数22作为参数调用 gdb_test() 函数
print gdb_test(a)：将以变量 a 作为参数调用 gdb_test() 函数
display 表达式：在单步运行时将非常有用，使用display命令设置一个表达式后，它将在每次单步进行指令后，紧接着输出被设置的表达式及值。如： display a
watch 表达式：设置一个监视点，一旦被监视的“表达式”的值改变，gdb将强行终止正在被调试的程序。如： watch a
whatis ：查询变量或函数
info function： 查询函数
扩展info locals： 显示当前堆栈页的所有变量

#### 6、查看运行信息

where/bt ：当前运行的堆栈列表；
bt backtrace 显示当前调用堆栈
up/down 改变堆栈显示的深度
set args 参数:指定运行时的参数
show args：查看设置好的参数
info program： 来查看程序的是否在运行，进程号，被暂停的原因。

#### 7、分割窗口

layout：用于分割窗口，可以一边查看代码，一边测试：
layout src：显示源代码窗口
layout asm：显示反汇编窗口
layout regs：显示源代码/反汇编和CPU寄存器窗口
layout split：显示源代码和反汇编窗口
Ctrl + L：刷新窗口
layout next：显示下一个layout
layout prev：显示上一个layout
Ctrl + x，再按1：单窗口模式，显示一个窗口
Ctrl + x，再按2：双窗口模式，显示两个窗口
Ctrl + x，再按a：回到传统模式，即退出layout，回到执行layout之前的调试窗口。
Ctrl + L：刷新窗口，每当窗口显示不正常的时候都可以使用此组合键刷新。











### 指令分类

#### 基础指令

>GDB基础调试命令总结
>
>s step，si步入
>
>n 执行下一条指令 ni步入
>
>b 在某处下断点，可以用
>
>b * adrress
>
>b function_name
>
>info b 查看断点信息
>
>delete 删除所有断点
>
>c 继续
>
>r 执行
>
>finish 执行完当前函数返回到调用它的函数。运行程序，直到当前函数运行完毕返回再停止。例如进入的单步执行如果已经进入了某函数，而想退出该函数返回到它的调用函数中，可使用命令finish.  
>
>(gdb) until 或(gdb) u  指定程序直到退出当前循环体这里，发现需要把光标停止在循环的头部，然后输入u这样就自动执行全部的循环了。
>
>(gdb) jump 5 跳转执行程序到第5行：这里，可以简写为"j 5"需要注意的是，跳转到第5行执行完毕之后，如果后面没有断点则继续执行，而并不是停在那里了。
>
>另外，跳转不会改变当前的堆栈内容，所以跳到别的函数中就会有奇怪的现象，因此最好跳转在一个函数内部进行,跳转的参数也可以是程序代码行的地址,函数名等等类似list。
>
>(gdb) return 强制返回当前函数: 这样，将会忽略当前函数还没有执行完毕的语句，强制返回。return后面可以接一个表达式，表达式的返回值就是函数的返回值。
>
>disass addr 查看addr处前后的反汇编代码
>
>disass functions 参看fucntion函数的反汇编代码
>
>p 系列
>
>p system/main 显示某个函数地址
>
>p $esp 显示寄存器
>
>p/x p/a p/b p/s。。。
>
>p 0xff - 0xea 计算器
>
>print &VarName 查看变量地址
>
>p * 0xffffebac 查看某个地址处的值



format的范围取值：

- x 按16进制格式显示变量
- d 按10进制格式显示变量
- u 按16进制格式显示无符号整型
- o 按8进制格式显示变量
- t 按2进制格式显示变量
- a 按16进制格式显示变量
- c 按字符格式显示变量
- f 按浮点数格式显示变量



#### x系列

```
x系列
命令格式：x/<n/f/u> <addr>
n是一个正整数，表示需要显示的内存单元的个数
 
f 表示显示的格式(可取如下值: x 按十六进制格式显示变量。d 按十进制格式显示变量。u 按十进制格式显示无符号整型。o 按八进制格式显示变量。t 按二进制格式显示变量。a 按十六进制格式显示变量。i 指令地址格式c 按字符格式显示变量。f 按浮点数格式显示变量。)
 
u 表示从当前地址往后请求的字节数 默认4byte,u参数可以用下面的字符来代替，b表示单字节，h表示双字节，w表示四字 节，g表示八字节
 
<addr>表示一个内存地址
 
x/xw addr 显示某个地址处开始的16进制内容，如果有符号表会加载符号表
 
x/x $esp 查看esp寄存器中的值
 
x/s addr 查看addr处的字符串
 
x/b addr 查看addr处的字符
 
x/i addr 查看addr处的反汇编结果
```



#### info系列

```
info系列
 
info register $ebp 查看寄存器ebp中的内容 (简写为 i r ebp)
 
i r eflags 查看状态寄存器
 
i r ss 查看段寄存器
 
i b 查看断点信息
 
i functions 查看所有的函数
 
disas addr 查看addr处前后的反汇编代码
 
stack 20 查看栈内20个值
 
show args 查看参数
 
vmmap 查看映射状况 peda带有
 
readelf 查看elf文件中各个段的起始地址 peda带有
 
parseheap 显示堆状况 peda带有
```

