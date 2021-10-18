# [ACTF新生赛2020] SoulLike

---

知识点：

- flag爆破



---



### 运行程序，查看特征

没什么新意 就是要输入 给反馈



### 查壳

无壳



### IDA静态分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/SoulLike_1.png)

由上图可知 该程序 前五个字符位为 **actf{** 然后将后序括号内字符复制送入Verify函数验证



Verify函数过长，无法反编译 也无意义，仔细查看，可得到下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/SoulLike_2.png)

由上图可知，该程序会告诉我们 输入的flag 错误的位置，而字符串长度为12 ，不算太差 所以可以爆破

### 脚本

```python
from pwn import *
import re

flag = "actf{"
# context.log_level="debug"

k = 0
while True:
    for i in range(33,127):
        p = process('./SoulLike')
        _flag = flag + chr(i)  
        print _flag
        p.sendline(_flag)             #发送一行数据，相当于在数据后面加\n
        s = p.recvline()			#接受一行数据
        r = re.findall("on #(.*?)\n", s)[0]
        r = int(r)
        if r == k:
            print 'no'
            print k
        elif r == k + 1:
            print s
            flag += chr(i)
            k += 1
            p.close()
        p.close()
    if k > 11:
        break
print flag
```

