# Easy_re

## 说明

- Perl 语言逆向

[Recover Perl from an EXE](https://metalamin.github.io/Perl-from-EXE-EN/)

## 可执行文件分析

分析字符串，可以指导该程序是`perl`打包形成`exe`文件；使用的`PerlApp`来制作的；如下图所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-27_19-09-44.png)

## Perl打包程序特点

到目前为止，**所有Perl到EXE打包程序都必须在启动解释器之前清楚地保存脚本。在某些情况下，它们存储在一个文件中，而在另一些情况下，它们保存在内存中。**在该样本中，`Perl`内存中解包。让我们看看使用`x64dbg`检索它的方法。



## 恢复Perl脚本

使用`x64dbg`加载程序，然后**右键->搜索->所有模块->字符串** ，查找`script`字符串，下断点

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-27_19-27-41.png)

两次`F9`运行，然后会停在断点处；

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-27_19-32-27.png)

得到以上信息；将内存区域信息转存即可得到`perl`脚本