# Ubuntu 20.04安装Pwn 环境

---

参考文献：

[Pwn环境与工具(较全)](https://www.yuque.com/u1499710/fgcf17/oo57bg?language=zh-cn#FzIno)

[Pwn 工具安装总汇(详细)](http://wafuter.jxustctf.top/2020/09/11/Pwn-%E5%B7%A5%E5%85%B7%E5%AE%89%E8%A3%85%E6%80%BB%E6%B1%87/)

[CTF_Pwn环境搭建](https://bbs.pediy.com/thread-257558.htm)

---

### Vim安装

```
sudo apt-get install vim
```

至于Vim的进一步配置，方法较多 就不详细展开了

### Pwntools安装

官方文档 安装：

```
apt-get update
apt-get install python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential
python3 -m pip install --upgrade pip
python3 -m pip install --upgrade pwntools
```

事实上 Ubuntu20.04 自带Python3  无Python2 环境

主要问题 在于后两步，使用官方的源（如果一开始切换了镜像源，可以直接使用上面的命令），下载超时无法成功，因此需要 使用 -i参数 指定为 清华的镜像源 ，命令如下

```
python3 -m pip install --upgrade pip -i https://pypi.tuna.tsinghua.edu.cn/simple
python3 -m pip install --upgrade pwntools -i https://pypi.tuna.tsinghua.edu.cn/simple
sudo apt-get update
```

安装成功验证

使用三条指令

```
python3
import pwn
pwn.asm("xor eax,eax")
```

效果如下图，无报错

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Pwn_2021-02-10_00-16-18.png)



### 在64位下运行32位程序

Ubuntu20.04 仅支持64位的 原始不支持32位程序运行，所以需要

```
sudo apt install libc6:i386
sudo apt install libstdc++6:i386
或者直接安装gcc-multilib解决问题（推荐使用此方法）
sudo apt install gcc-multilib
```



### git，gdb 和 gdb-multiarch， binfmt 安装

gdb用于调试程序，gdb-multiarch是一个可以调试各种架构的gdb，就不需要自行编译target为不同架构的gdb了，十分方便。同时安装 binfmt 用来识别文件类型，然后传递其至相对应架构的qemu虚拟机。

```
sudo apt-get install git gdb gdb-multiarch
sudo apt-get install "binfmt*"
```



### pwndbg

peda/pwngdb/gef，这是三个常见的gdb的三个插件，配合gdb使用可以提升调试效率

```
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```

在最后一步运行脚本时，可能下载包超时，需要替换 setup.sh 的源 ，使用 root权限 用vim 编辑该脚本

编辑setup.sh
临时修改pip的源并设置超时时间。

```
# Install Python dependencies
${PYTHON} -m pip --default-timeout=60 install ${INSTALLFLAGS} -Ur requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

测试安装是否成功

shell中输入gdb能够看到pwndbg>,如下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Pwn_2021-02-10_14-31-18.png)



**Attention： Pwndbg 安装完成后需要使用root权限才会启动pwndbg，否则只是一般的gdb**



### LibSearcher

Python3版本

https://pypi.org/project/LibcSearcher/

```
pip3 install LibcSearcher
```



### QEMU

一般的题用不到QEMU，因此尚未安装，同时QEMU和KVM的关系与差别 有待学习了解 ，所以先放放



### tmux

[Tmux 使用教程 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2019/10/tmux.html)

一个终端分屏，便于调试

```shell
sudo apt-get install tmux   #安装tmux
context.terminal = ['tmux','splitw','-h']  #exp的配置，需要在tmux运行 exp
```

配置tmux鼠标操作

- 在Terminal输入`vi ~/.tmux.conf`创建一个配置文件, 并在文件中输入`set -g mouse on`并保存.
- 随后在Terminal中输入`tmux`进入tmux模式, 进入后按`Ctrl+b`+`:`, 此时tmux底部会变颜色. 在此输入`source ~/.tmux.conf`并回车即可.

[配置tmux鼠标操作_DY的博客-CSDN博客_tmux 鼠标](https://blog.csdn.net/weixin_41677877/article/details/90004300)

