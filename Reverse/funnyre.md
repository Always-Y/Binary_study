# [WUSTCTF2020]Funnyre

**花指令+Angr**

[buuctf刷题记录25 [WUSTCTF2020\]funnyre_ytj00的博客-CSDN博客](https://blog.csdn.net/ytj00/article/details/107735151)

[funnyre-花指令与符号执行angr](https://0xkami.top/2021/01/26/funnyre-%E8%8A%B1%E6%8C%87%E4%BB%A4%E4%B8%8E%E7%AC%A6%E5%8F%B7%E6%89%A7%E8%A1%8Cangr/#more)

字符串是 flag{32个字符}

处理后为

0xd9,0x2c,0x27,0xd6,0xd8,0x2a,0xda,0x2d,0xd7,0x2c,0xdc,0xe1,0xdb,0x2c,0xd9,0xdd,0x27,0x2d,0x2a,0xdc,0xdb,0x2c,0xe1,0x29,0xda,0xda,0x2c,0xda,0x2a,0xd9,0x29,0x2a,





```python
from idc_bc695 import *
print("start:")
for i in range(32):
    print(hex(Byte(0x000004025C0+i)),end=",")

```





### 符号执行

```python
import angr
import sys
import claripy

def main(argv):
    path_to_binary = "funnyre"
    Proj = angr.Project(path_to_binary)

    state = Proj.factory.entry_state(addr = 0x400605)
    flag = claripy.BVS('flag',8*32)

    state.memory.store(0x603055+0x400+5,flag)   #这是一个动态分配，在堆中，所以只需要找一个任意地址能够存储向量即可
    state.regs.rdx = 0x603055+0x400      #后面用到了 rdx 和 rdi
    state.regs.rdi = 0x603055+0x400+5

    sm = Proj.factory.simgr(state)

    print("Ready")

    sm.explore(find = 0x401DAE)

    if sm.found:
        print("success")
        x=sm.found[0].solver.eval(flag,cast_to = bytes)
        print(x)
    else:
        print("error")

if __name__ =='__main__':
    main(sys.argv)

```

