# Ubuntu20.04 配置 VScode

### 下载VScode

在Ubuntu商店即可下载，或者通过命令下载



### 安装插件

打开VScode 通过插件选项 安装 以下三个插件：

- C/C++
- Code Runner
- c-cpp-compile-run



### 配置环境

打开 File->preference->settings 

在搜索框中 输入runInTerminal，勾选如下图所示

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/VScode_2021-02-13_23-57-57.png)



### 配置文件夹

程序运行需要一个文件夹 按提示配置即可

[Ubuntu18 VsCode搭建C++编译环境(已测试) | 大专栏 (dazhuanlan.com)](https://www.dazhuanlan.com/2019/10/21/5dad476db556f/)



### VScode 快捷键

F5 运行程序 

F1 搜索终端



### Python环境配置

在ubuntu20.04中 使用的是python3，但是 code runer 插件 默认使用Python 命令去运行.py脚本,如下图所示：

```
python -u "/home/always/PWN_Study/Payload/Test.py"  #会报错，如下图所示
```

>Command 'python' not found, did you mean:
>
>  command 'python3' from deb python3
>  command 'python' from deb python-is-python3



因此 我们需要修改默认的运行指令，改为Python3 xx.py  

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-24_19-34-50.png)

选择 Code Runner-> Extension Settings

找到Executor Map，点击 `在settings.json中编辑`

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-02-24_19-36-55.png)

如上图所示，将对应的Python 改为python3 即可，然后 `ctrl`  + s 保存即可



### Python3 和 Python2共存

[VSCode切换Python2和Python3](https://blog.csdn.net/nimble22/article/details/107607314)

​		因为电脑上可能Python2 和Python3 共存，所以所以设置 Python即为运行 Python3 ，使用Python2 才是表示 运行Python2 在VScode中默认的运行指令是 `python -u` 只能正确执行Python3 对 Python2 有问题 所以要修改

```
"python": "$pythonPath $fullFileName"
```





登录 用户密码 123456 
root 密码 1234qwer
已更换软件镜像源
安装了32位程序支持环境
