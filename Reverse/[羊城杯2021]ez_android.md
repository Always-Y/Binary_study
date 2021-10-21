# ez_android

## 登录页面

- 首先是使用`jadx`对软件进行反编译，如下图所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-25_15-28-19.png)

- 找到程序的主入口

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-25_15-51-56.png)

- 对主入口函数进行分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-25_16-00-17.png)

- 在对应的资源文件中查看相关数据值 

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-25_16-02-15.png)

同样 在该文件中查找 可知  `encode`的编码方式为`md5`

由此可以知道一下几点

- 用户名是 `admin` 
- 输入的密码 处理后 的值是 `c232666f1410b3f5010dc51cec341f58`
- 密码的处理流程 就是 `首先计算md5值 按字节划分，每字节的值 -1`

从而得到 密码 `md5`值为`c33367701511b4f6020ec61ded352059`，在线解密可得到密码为`654321`

输入到登录页面，会发生跳转，由于比赛已经结属，所以在此为静态分析



## 跳转页面

- 跳转页面活动如下

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-25_16-14-24.png)



`key`:`TGtUnkaJD0frq61uCQYw3-FxMiRvNOB/EWjgVcpKSzbs8yHZ257X9LldIeh4APom`

- 查看`CheckFlagActivity`

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-25_16-17-31.png)

由上图可以指导该页面主要的活动是 `checkFlag`函数，同时从资源文件得到`encodeflag`:`3lkHi9iZNK87qw0p6U391t92qlC5rwn5iFqyMFDl1t92qUnL6FQjqln76l-P`

- 进一步查看 `EncodeUtils`函数

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-25_16-21-27.png)

由此可以推理得到整个流程：

- 从服务器得到的字符串为编码表
- `flag` 通过 换表的`base64`编码 得到`encodeflag`



## 整个流程

首先是登录界面：

- 用户名是 `admin` 
- 输入的密码 处理后 的值是 `c232666f1410b3f5010dc51cec341f58`
- 密码的处理流程 就是 `首先计算md5值 按字节划分，每字节的值 -1`

从而得到 密码 `md5`值为`c33367701511b4f6020ec61ded352059`，在线解密可得到密码为`654321`

然后是`flag`验证：

- 登录成功后，会得到一个字符串，该字符串为变种`base64`编码的编码表

- `flag`通过变种`base64`编码得到`encodeflag`

  

## 解密脚本

```python
enc_md5 = bytearray.fromhex('c232666f1410b3f5010dc51cec341f58')
for i in enc_md5:
  print(hex(i+1).replace("0x",''),end='')
print("\n")
 
import base64
base1 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
t='3lkHi9iZNK87qw0p6U391t92qlC5rwn5iFqyMFDl1t92qUnL6FQjqln76l-P'
base = 'TGtUnkaJD0frq61uCQYw3-FxMiRvNOB/EWjgVcpKSzbs8yHZ257X9LldIeh4APom '
ans = ''
for i in t:
    ans += base1[base.index(i)]
print(ans)
print(base64.b64decode(ans))
#c33367701511b4f62ec61ded352059
#SangFor{212f4548-03d1-11ec-ab68-00155db3a27e}
```

