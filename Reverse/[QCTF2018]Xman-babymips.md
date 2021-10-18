# [QCTF2018]Xman-babymips

总的处理过程为两部分：

- 对输入的32个字符进行逐字符异或运算
- 对后27个字符根据奇偶项在进一步做处理

---

### Die 查看信息

程序就说了，是mips程序，Linux无法运行，直接拖入IDA





### 静态分析



#### 验证循环

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-06-16_15-47-38.png)

上图标注的一行，推测为对输入字符串的逐字符异或，但解析为&i，所以根据前五个字符结果使用一下程序验证

验证程序的循环

```
str = [0x51,0x7c,0x6a,0x7b,0x67]
for i in range(5):
    print(chr(str[i]^(32-i)),end=",")
```

输出为

>q,c,t,f,{,    ---->可知该循环为异或



### 后27个字符的进一步处理

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-06-16_15-55-52.png)

感觉逆向算法较为麻烦，因此使用ZZ3求解器求解

#### 输出IDA数据

脚本

```python
from idc_bc695 import *
for i in range(27):
    print(hex(Byte(0x400B98+i)),end=",")
    
#0x52,0xfd,0x16,0xa4,0x89,0xbd,0x92,0x80,0x13,0x41,0x54,0xa0,0x8d,0x45,0x18,0x81,0xde,0xfc,0x95,0xf0,0x16,0x79,0x1a,0x15,0x5b,0x75,0x1f,
```



#### Z3求解脚本

所以先对 经过异或运算后的27个字符的值建立 8bit的向量，然后约束求解，求解脚本如下：

```python
from z3 import *

#最后的数据为data 
data = [0x52,0xfd,0x16,0xa4,0x89,0xbd,0x92,0x80,0x13,0x41,0x54,0xa0,0x8d,0x45,0x18,0x81,0xde,0xfc,0x95,0xf0,0x16,0x79,0x1a,0x15,0x5b,0x75,0x1f]


#定义z3变量,每一个变量为8bit 共27个 ;求出hou27个字符异或运算后的值
Input =[BitVec('Input[%s]'%i,8) for i in range(27)]

# 求解器
s = Solver()

for i in  range(27):
    if (i+5)&1 != 0:  #分奇偶项设置约束条件
        s.add(((Input[i] >> 2) | (Input[i] << 6))==data[i])
    else:
        s.add(((4 * Input[i]) | (Input[i] >> 6))==data[i])

if s.check() == sat:
    m = s.model()
    for i in range(27):
        print(chr(m[Input[i]].as_long()^(27-i)),end="")  #中间插入^(27-i) 为前面异或的逆运算
        # print(hex(m[Input[i]].as_long()),end=",")
```

> ReA11y_4_B@89_mlp5_4_XmAn_}   #输出结果





### Flag

将两次的结果结合，前五个字符加上后五个字符，共32 个  即为**qctf{ReA11y_4_B@89_mlp5_4_XmAn_}，在BUU提交时，需要将qctf替换为flag！！！**