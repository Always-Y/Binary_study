# [GKCTF 2021]Crash

## 查壳

无壳，为64bit

## 程序运行

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-15_19-37-17.png)

## go程序特点

### go程序识别

go程序函数多，体积大；具有特别的区段`gopclntab`，相较于一般程序有极大区别，如下图所示；

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-15_19-26-38.png)



### go程序逆向

对于`go`程序与普通的程序不同，目前主要面对的有两种：

- 未脱去符号表的，具有函数名称

- 脱去符号表的，函数以地址表示(如上图所示)

  在这两者之中，相对来说，未脱去符号表的相对简单；我们只需要找到`main_main`函数即可，`go`程序从该函数入手；而对于脱去了符号表的函数，首先第一件事就是**恢复符号表**；这主要原理是根据`.gopclntab`节来得到的(具体原理，再找时间写个文档)；目前主要使用的工具是`IDAGolangHelper`来实现的；

在`IDA`左上角，**file->script file ->选择IDAGolangHelper对应文件夹 ->Goutils.py**

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-15_19-41-38.png)



### go程序字符串

**`go`程序字符串与我们在一般程序的字符串存储方式不同，在一般程序中，字符串以`\0`为结束，所以`IDA`根据该原理识别字符串；但`go`程序并不以`\0`作为结尾，其在内存中存放分为两部分，如下所示：**

```c
type StringHeader struct {

  Data uintptr      // 字符串首地址

  Len  int        // 字符串长度

}
```

**因此IDA很难正确识别出Go程序的字符串**

由下图，可以看到，查找程序运行的提示信息`please`无结果，IDA无法正确识别该字符串

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-15_19-38-22.png)

实际存储效果如下图所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-15_19-45-22.png)





## 程序逆向

> 整个程序其实很简单，主要就是经过3DES，sha256，sha512和md5四个算法
>
> flag长度要求为43byte，格式是GKCTF{}，所以中间一共是36个字符串；3DES计算得到前24位；后面的三种hash算法分别计算 4个字符的hash值



### 3DES

分析程序易得，可知为CBC模式，

```
"key": "WelcomeToTheGKCTF2021XXX",',0Dh,0Ah
 "iv": "1Ssecret"',0Dh,0Ah
```

字符串加密后计算base64得到如下字符串

> o/aWPjNNxMPZDnJlNp0zK5+NLPC4Tv6kqdJqjkL0XkA=

```python
#获取base64编码值
for i in range(44):
    print(chr(Byte(0x5507AF+i)),end="")
print("success")
```

在线解密即可得到如下所示：

flag 长度 为43

```
87f645e9-b628-412f-9d7a-        32  6-30  24byte
```

可以知道还缺12个字符。



### Hash计算

12个字符分为三组，每组四个字符，进行计算，然后与特定hash值对比，因此可以爆破求解，代码如下

> sha256  hex 得到 6e2b55c78937d63490b4b26ab3ac3cb54df4c5ca7d60012c13d2d1234a732b74            ------ >e402

> sha512 hex 得到  6500fe72abcab63d87f213d2218b0ee086a1828188439ca485a1a40968fd272865d5ca4d5ef5a651270a52ff952d955c9b757caae1ecce804582ae78f87fa3c9   -----> f20a



> md5   ff6e2fd78aca4736037258f0ede4ecf0     ----------    > f940

 









```python
import hashlib

s = "6500fe72abcab63d87f213d2218b0ee086a1828188439ca485a1a40968fd272865d5ca4d5ef5a651270a52ff952d955c9b757caae1ecce804582ae78f87fa3c9"


def sha256_Crash():
    for a in range(32,128):
        for b in range(32,128):
            for c in range(32,128):
                for d in range(32,128):
                    str = chr(a) + chr(b) +chr(c) +chr(d)
                    h = hashlib.sha256(str.encode("utf-8")).hexdigest()
                    if h == s:
                        print(str) 
                        print("Over")
                        return 0

def sha512_Crash():
    for a in range(32,128):
        for b in range(32,128):
            for c in range(32,128):
                for d in range(32,128):
                    str = chr(a) + chr(b) +chr(c) +chr(d)
                    h = hashlib.sha512(str.encode("utf-8")).hexdigest()
                    if h == s:
                        print(str) 
                        print("Over")
                        return 0

sha512_Crash()
```



### Attention

**IDA7.5对go程序的反汇编效果并不是特别好，会有一定的信息缺失，所以我们还是需要多看汇编，例如在本题中，计算所得到的hash字符串静态存储，但是IDA反汇编的代码中很难得到，其次对于MD5的计算，ida反汇编直接忽视了，所以我们还是需要根据汇编来看**

反汇编效果：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-15_19-57-45.png)

原始汇编效果：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-10-15_19-59-06.png)



## flag

GKCTF{87f645e9-b628-412f-9d7a-e402f20af940}
