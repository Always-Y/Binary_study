# Ubuntu 使用问题集锦



---

### 新安装的Ubuntu 20.04 不能运行 二进制程序

A：注意查看 程序的 位数，是否为32位程序，Ubuntu为64位，原始状态 无法执行32位程序，需要安装一些环境

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2020-12-16_17-43-41.png)

解决命令：

```
sudo apt-get update
sudo apt-get purge libc6-dev
sudo apt-get install libc6-dev
sudo apt-get install libc6-dev-i386
```

参考：[64位ubuntu下运行32位程序](https://blog.csdn.net/u013112749/article/details/89921308)



### 设置root用户

第一使用root用户前，需要设置root 用户密码，使用如下指令设置

```
sudo passwd root   #ubuntu20.10 要求密码8位以上，且存在密码检查,过于简单则无法通过(1234qwer)
```





### 切换下载镜像源

一般Ubuntu的默认镜像下载源为官方下载源，但由于墙的存在，下载速度较慢。因此需要修改为国内的下载镜像源网址。



