# [羊城杯 2020]easyre

### 运行程序

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-06-16_17-09-33.png)

基本就是输入，然后提示输入是否正确



### Die 查看

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-06-16_17-11-05.png)

64 位程序，无壳



### 静态分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-06-16_19-40-21.png)

 

```


import base64
#最后的数据为data 
str = "EmBmP5Pmn7QcPU4gLYKv5QcMmB3PWHcP5YkPq3=cT6QckkPckoRG"
data = [ord(c) for c in str]
s =""

for i in range(52):
    if data[i] >=0x41 and data[i] <= 0x5A :
        data[i] = (data[i] - 65 - 3) % 26 + 65
    elif data[i] >=0x61 and data[i] <= 0x7A :
        data[i] = (data[i] - 97 - 3) % 26 + 97
    elif data[i] >=0x30 and data[i] <= 0x39 :
        data[i] = (data[i] - 48 - 3) % 10 + 48
    else :
        pass

for _ in data:
    s += chr(_)
print(s)
s = s[13:26] + s[39:52] + s[0:13] + s[26:39]  #交换字符串顺序


print(base64.b64decode(s))  #base64解码
```

