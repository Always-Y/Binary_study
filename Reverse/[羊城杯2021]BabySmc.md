# BabySmc

## 静态分析

- 看题目就知道 具有 Smc(Self modified code) 处理

> 运行程序查看特征

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-18_16-04-38.png)

> 把程序拖入IDA 进行分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-18_16-07-01.png)

> 所以推测到 lpAddress即为需要解析的地址；**解密 长度为 (0x1D00 - 0x1085) ** = 3195
>
> 查找对应的函数，**SMC解密代码的特征标志就是 VirtualProtect函数，由于未解密前的数据以二进制可读形式存在，其所在页只有可读权限，因此在解密的之前，需要通过该函数修改所在页的的权限，使其可执行，可写**

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-18_16-09-12.png)

> 逐位进行 循环右移，然后与0x5a 异或运算



## SMC解密代码

两种解决方案：

- 分析IDA得到解密函数，通过IDApython编写解密函数；静态解密
- 程序在运行过程中；会进行解密，所以只要打下合适的断点，即可得到代码

这次主要介绍 动态调试的方法

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-18_20-07-38.png)

解密后，调整数据，然后进行分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-18_20-09-00.png)



## 反汇编分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-18_20-09-00.png)

分析反汇编代码



![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-18_20-11-39.png)

打断点，动态运行至上述断点处，然后得到`64byte`数据

```
0xe4,0xc4,0xe7,0xc7,0xe6,0xc6,0xe1,0xc1,
0xe0,0xc0,0xe3,0xc3,0xe2,0xc2,0xed,0xcd,
0xec,0xcc,0xef,0xcf,0xee,0xce,0xe9,0xc9,
0xe8,0xc8,0xeb,0xcb,0xea,0xca,0xf5,0xd5,
0xf4,0xd4,0xf7,0xd7,0xf6,0xd6,0xf1,0xd1,
0xf0,0xd0,0xf3,0xd3,0xf2,0xd2,0xfd,0xdd,
0xfc,0xdc,0xff,0xdf,0x95,0x9c,0x9d,0x92,
0x93,0x90,0x91,0x96,0x97,0x94,0x8a,0x8e,
```

往下继续分析，可知3个为一组进行编码，当剩余一个字符时，会进行填充字符`1`和字符`4`；若是剩余两个字符则填充字符`4`

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-18_20-35-07.png)

**综上，得到整个流程类似于Base64；编码表为上诉自己定义的数据表，然后类似base64的编码过程，每三个一组索引得到四个数据然后与特定数据进行异或操作；最后模仿base64编码不足，填充字符`1`和字符`4`;**

## 解密代码

> 获取 编码表数据

```python
for i in range(64):
    if i%8==0:
        print("")
    print(hex(Byte(0x0D252B1FB10+i)),end=",")
```



> 通过对应码表替换 

```c++
#include <iostream>
#include <string>

using namespace std;

void Decode(string str);


string enstr="ABCDEFGHIJKLMNOPQRSTUVWXYZ"
             "abcdefghijklmnopqrstuvwxyz"
             "0123456789+/";

unsigned char Table[64]={
0xe4,0xc4,0xe7,0xc7,0xe6,0xc6,0xe1,0xc1,
0xe0,0xc0,0xe3,0xc3,0xe2,0xc2,0xed,0xcd,
0xec,0xcc,0xef,0xcf,0xee,0xce,0xe9,0xc9,
0xe8,0xc8,0xeb,0xcb,0xea,0xca,0xf5,0xd5,
0xf4,0xd4,0xf7,0xd7,0xf6,0xd6,0xf1,0xd1,
0xf0,0xd0,0xf3,0xd3,0xf2,0xd2,0xfd,0xdd,
0xfc,0xdc,0xff,0xdf,0x95,0x9c,0x9d,0x92,
0x93,0x90,0x91,0x96,0x97,0x94,0x8a,0x8e};

unsigned char num[4] ={0xA6,0xA3,0xA9,0xAC};            //注意 单字节的异或运算 需要使用 无符号char型

char find_ch(unsigned char n){      //传参也需要是unsigned char 类型 否则会有数据缺失
    for(int i=0;i<64;i++){
        if(Table[i] == n)
            return enstr[i];
    }
    return '=';
}


void Decode(string str){
    string s ="";
    unsigned char ch;
    for(int i=0;i<str.size();i++){
        ch = str[i]^num[i%4];
        s += find_ch(ch);
    }
    cout<<s;
}


int main(){
    string str = "H>oQn6aqLr{DH6odhdm0dMe`MBo?lRglHtGPOdobDlknejmGI|ghDb<4";            //56个字符
    Decode(str);
    return 0;
}
```

得到编码数据：**U2FuZ0ZvcntYU0FZVDB1NURRaGF4dmVJUjUwWDFVMTNNLXBaSzVBMH0=**



## Flag

在线 解密后 得到Flag 为 **SangFor{XSAYT0u5DQhaxveIR50X1U13M-pZK5A0}**；
