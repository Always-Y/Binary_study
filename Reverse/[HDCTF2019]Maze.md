# [HDCTF2019]Maze

---

- 花指令的处理
- [IDA学习经验和实战记录–手过花指令(1)](https://blog.csdn.net/weixin_44352049/article/details/85567929)
- [IDA学习经验和实战记录–手过花指令(2)](https://blog.csdn.net/weixin_44352049/article/details/85850733)

---

### 运行程序查看特征

运行程序，得到如下图所示：

![图1](D:\Binary_Study\[HDCTF2019]Maze\1.png)

由上图可知，给了提示字符串，告诉我们穿越迷宫来得到flag，随意输入后，回车，界面框消失，程序终止



### 程序查壳

惯例操作，使用detectItEasy查壳，如下图所示：

![图2](D:\Binary_Study\[HDCTF2019]Maze\2.png)

由图知，该程序加了UPX壳，程序使用32位的终端界面程序，使用C/C++来编写。

脱壳方式可以 手动脱壳，或者使用 软件进行脱壳，由于本题重点不在脱壳，所以直接使用UPX脱壳工具，进行脱壳，不再过多的叙述

脱壳后查壳 如下图所示：

![图3](D:\Binary_Study\[HDCTF2019]Maze\3.png)



### IDA静态分析

将程序拖入IDA 得到如下图所示：

![图3](D:\Binary_Study\[HDCTF2019]Maze\4.png)

由上图，可以看到，有大量代码，IDA无法识别，且部分代码,IDA标红，较为奇怪，这个涉及到了**花指令**

首先，标注一下，IDA的几个指令 ，D 将程序以原始数据展示，C将数据解读为代码展示



#### 花指令的处理

仔细观察，主要是可以发现，上图中，有两处指令，较为奇怪，如下图所示：

![图5](D:\Binary_Study\[HDCTF2019]Maze\5.png)

对于花指令的处理，也很简单，直接NOP掉，第一处的条件无价值，所以通过IDA选择该指令，右键->keypatch->patcher,然后更改为nop，如下图所示：

![图6](D:\Binary_Study\[HDCTF2019]Maze\6.png)

通过patcher修改后的，IDA会对其进行备注；

对于第二次call 指令的地址，明显过大，不符合常理，应该不存在，所以选择通过D指令，将其变化为数据，如下图：

![图7](D:\Binary_Study\[HDCTF2019]Maze\7.png)

对于转换后的数据，查看机器码的对照表，可知 E8为call指令 ，58为pop指令，所以可以推测该call指令是多余的，故意插入的花指令，所以尝试将他nop掉，如下图所示，将其nop后，其余部分自动识别为新的正常代码

![图8](D:\Binary_Study\[HDCTF2019]Maze\8.png)



前端标红，表示该代码没有自动被IDA识别为函数，所以选中这些代码，P键将其标识为函数，如下图：

![图9](D:\Binary_Study\[HDCTF2019]Maze\9.png)

此时F5 反编译可得：

![图10](D:\Binary_Study\[HDCTF2019]Maze\10.png)

由图可知，通过awsd四个键来进行上下左右移动；

从数据中整理得到迷宫，如下图

![图11](D:\Binary_Study\[HDCTF2019]Maze\11.png)

由+号出发到达F  得到flag 为**flag{ssaaasaassdddw}**