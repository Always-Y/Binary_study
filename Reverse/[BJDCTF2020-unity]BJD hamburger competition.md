# [BJDCTF2020]BJD hamburger competition

---

- unity3D 游戏的逆向，第一次接触
- 文件较多，搞不清重点
- .NET  C#

Q:如何定位逆向的关键文件？

Q:如何在关键文件中确定关键的位置

---

### 运行程序，获取信息

效果如下图所示：

![图1](D:\Binary_Study\[BJDCTF2020-unity]BJD hamburger competition\1.PNG)



整个过程，就是一个选食材做汉堡的，然后品尝的游戏；没有提到任何关于flag的信息

推测： 根据游戏的流程，可以推测，可能与做汉堡的素材等相关

由于unity3D程序，素材多，是一个大的文件夹，开始前，开始前我们首先需要定位重点需要关注的文件和函数，否则内容太多，一头雾水。

**按照一般来说，程序都是分析运行的.exe，但是unity却不一样，他需要分析的是/Data/Managed/Assembly-CSharp.dll文件，因为unity中，C#编写的主逻辑模块代码静态编辑之后存储于Assembly-CSharp.dll文件中**



### 查壳

![图2](D:\Binary_Study\[BJDCTF2020-unity]BJD hamburger competition\2.PNG)



由上图可知，该DLL为32 ，.NET程序，所以使用dnSpy32分析；

### 逆向分析

如下图所示，在dnSpy中加载程序后，使用关键字搜索，输入游戏中的文字元素，来定位关键函数：

![图3](D:\Binary_Study\[BJDCTF2020-unity]BJD hamburger competition\3.png)

点击，跳转到如下图所示：

![图3](D:\Binary_Study\[BJDCTF2020-unity]BJD hamburger competition\4.PNG)

由上图，整个流程，流程相当，清晰明了，除了汉堡底和汉堡顶需要四种素材，这四种素材会使secre的值转化为字符串后取hash值等于一个固定的值，然后再取该字符串的MD5值；



在这里，由于使用解密网站可以将该SHA1值直接解出为字符串1001；所以可以直接用于计算MD5

**要注意，在改函数中MD5函数是自己编写的，仅取了前20位**。