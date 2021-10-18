## V&N CSRe --2020  -----BUUCTF

该题型为.NET，第一次遇到，做一个总结

---

参考文档：  C#

[.Net零基础逆向教程](https://www.muruoxi.com/jiaocheng/4052.html)

工具：

De4dot

dnSpy

---

### 查壳

---

使用DetectItEasy查壳  如下图所示：

![图1](D:\Binary_Study\V&N-2020-CSRe\1.PNG) 

由上图可知以下几点：

- 该软件无壳
- 该程序由.NET编写
- 该程序为32位

### 反汇编

---

针对.NET程序的反汇编 需要使用dnSpy 软件  将 该程序拖入 dnSpy 可得到下图：

![图2](D:\Binary_Study\V&N-2020-CSRe\2.png)

由图可知，该程序入口点为不规则字符串，可能存在混淆

展开程序目录 选择PE->右键->转到入口点

![图三](D:\Binary_Study\V&N-2020-CSRe\3.png)

得到如下图所示 ，发现有许多乱码

![图四](D:\Binary_Study\V&N-2020-CSRe\4.png)



**由此推测存在代码混淆**



### 代码混淆查证

---

使用exeinfo PE查看

![图5](D:\Binary_Study\V&N-2020-CSRe\5.png)

百度可知  Eazfuscator.NET 是一种.NET平台的混淆器，证实了 程序存在代码混淆



### 反混淆

---

反混淆主要使用 de4dot工具  (注意使用最新版，要不然效果很差)

> de4dot则是一个开源的.Net反混淆脱壳工具，支持一键脱去Xenocode、.NET Reactor、MaxtoCode、Eazfuscator.NET、Agile.NET、Phoenix Protector、Manco Obfuscator 、CodeWall、NetZ .NET Packer 、Rpx .NET Packer、Mpress .NET Packer、ExePack .NET Packer、Sixxpack .NET Packer、Rummage Obfuscator、Obfusasm Obfuscator、Confuser 1.7、Agile.NET、Babel.NET、CodeFort、CodeVeil、CodeWall、CryptoObfuscator、DeepSea Obfuscator、Dotfuscator、 Goliath.NET、ILProtector、MPRESS、Rummage、SmartAssembly、Skater.NET、Spices.Net 等常见的壳

#### 使用方法：

1. 在CMD中使用 命令 脱壳，会得到一个  **原始文件名-cleaned** 的文件， 如下图：

![图6](D:\Binary_Study\V&N-2020-CSRe\6.png)



2. 也可以通过 按住鼠标左键，将我们要脱壳的程序直接拖到de4dot.exe上

   ![图7](D:\Binary_Study\V&N-2020-CSRe\7.png)
   
   

### 逆向求解

---

> 首先介绍，对于.NET的反编译主要使用 dnspy 软件

加载后如下图所示：

![图8](D:\Binary_Study\V&N-2020-CSRe\8.png)



可知 程序的入口点位 Class3.Main 	或者 对PE 右键 选择 转到入口点 进入主函数，得到下图

![图9](D:\Binary_Study\V&N-2020-CSRe\9.png)

可知关键函数为 Class3.smethod_0,进行字符串对比，点击进入可得到下图

![图10](D:\Binary_Study\V&N-2020-CSRe\10.png)

综合两部分  可以看出  **第45行代码，第一部分先是str，前面加上字符‘3’，后面加上字符‘9’，然后算哈希，与给的长字符串进行对比，看是否相等**

对于 50-51行代码仔细分析，发现实现的是与45行代码功能相同

所以对 所给的字符串SHA1解密，分别得到 字符串 “314159“ 和 ：”return“

然后去掉 3 和9 以及 re

可得**flag{1415turn}**