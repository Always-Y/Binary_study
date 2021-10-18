# IDA 相关资料整合

### IDA -XREF(交叉引用)概述

交叉引用分为数据交叉引用和代码交叉引用；

详见[IDA交叉引用](https://blog.csdn.net/hgy413/article/details/50594320)





### IDA 使用技巧总结

[IDA使用技巧](https://xz.aliyun.com/t/4205)



### 重要函数

MakeCode(ea) **#分析代码区，相当于ida快捷键C**

ItemSize(ea) **#获取指令或数据长度**

GetMnem(ea) **#得到addr地址的操作码**

GetOperandValue(ea,n) **#返回指令的操作数的被解析过的值**

PatchByte(ea, **value**) **#修改程序字节**

Byte(ea) **#将地址解释为Byte**

MakeUnkn(ea,0) **#MakeCode的反过程，相当于ida快捷键U**

MakeFunction(ea,end) **#将有begin到end的指令转换成一个函数。如果end被指定为BADADDR（-1），IDA会尝试通过定位函数的返回指令，来自动确定该函数的结束地址**



### API迁移兼容

目前使用的事`IDA7.5`版本，相对于`IDA7.0`其API做了一定的改变有所迁移，

[关于IDA7.5 IDApython api差异问题及解决办法 - 『脱壳破解区』 - 吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn](https://www.52pojie.cn/thread-1403005-1-1.html)

首先是官方对于api的详细迁移文档
https://www.hex-rays.com/products/ida/support/ida74_idapython_no_bc695_porting_guide.shtml
有了这个表，一些wp里的旧版代码搜索一下就能在7.5版使用了。



以下是兼容办法
**方法1**：
其次是在比较方便的在7.5用7.0API办法，来源：https://github.com/0xgalz/Virtuailor/issues/8
在代码面前：

> from idc_bc695 import *    即可。


**方法2**(一劳永逸版)：
来源官方文档：https://www.hex-rays.com/products/ida/support/ida74_idapython_no_bc695.shtml
找到

- ~/.idapro/cfg/idapython.cfg (Linux)
- %APPDATA%\Hex-Rays\IDA Pro\cfg\idapython.cfg (Windows) (绿色版的话在程序目录下那个cfg文件夹里)



把AUTOIMPORT_COMPAT_IDA695改为YES

