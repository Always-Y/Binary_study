# Angr----基于符号执行的二进制程序分析框架

---

参考资料：

[angr 系列教程(一）核心概念及模块解读](https://xz.aliyun.com/t/7117#toc-27)



### Angr简介

---

引用Angr开发者的话;

>angr 是一个python编写的二进制分析框架，它结合了静态与动态符号分析，适用于各种任务。

> 程序正常执行过程是将instruction逐条送入CPU执行，操作内存和设备的过程。而angr模拟执行的过程恰似人类去阅读代码，一边阅读的过程一边做预测、抽象执行结果，预估“执行结果”。这个预估过程就是一个symbolic execution的过程。在angr模拟执行前，必须先将变量（内存或者寄存器）符号化（声明哪些变量是符号），这些符号化的输入也正是程序的输入抽象。模拟执行过程的目标是预测“执行结果”，当输入不影响输出结果的时候，符号化的输入不会影响到输出。但是输入影响输出结果，那么符号化的输入一定是通过指令执行影响了数据流或者控制流。符号随着指令进行传播的过程，可以看做由符号和运算符构成的表达式(expression)，遇到条件分支的时候，表达式和关系符号共同形成一个约束（constraint）。理想情况下，一个程序的输出可以视作由输入符号和运算关系构成的表达式，以及各种条件约束：例如 当x<1且x>0时，y=x^2，其中 0<x<1为约束，x^2为表达式。所以angr的预期返回结果是”在XXX定义域下，会得到一个由输入做XXXXX变换得到的结果“。



>angr重要的数据结构包括simstate、simmanage、simsolver等等。其中我们要认清概念——

- 状态SimulateState
- 模拟管理器SimulatorManager

> SimulatorManager是angr模拟执行的控制中心，提供了.run , .explore, .step 等不同类型的执行选项。
>
> SimulateState则是angr的最小分析单元，对应每个step都有一个state，每个[http://state.solver.XXX](https://link.zhihu.com/?target=http%3A//state.solver.XXX)都记录了当前符号执行记录的expr和constraint (state.solver.constraints)（这里expr怎么导出还不确定）。
>
> 模拟执行到相应的条件分支的时候，会用claripy(z3的angr封装)去checkSatiesfied或者eval求解。也就是我们常用到的求解相应输出的输入了。



### Angr安装

---

Ubuntu环境下

参见[ubuntu安装python虚拟环境并以安装angr为例](https://blog.csdn.net/u012763794/article/details/106471316)





### 符号执行

#### 什么是符号？

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/SY_1.png)





#### 什么是执行路径？

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/SY_2.png)