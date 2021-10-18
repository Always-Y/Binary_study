# Nicepy

## 提取文件

反编译 `main.py`

```python
#!/usr/bin/env python
# visit https://tool.lu/pyc/ for more information
import util
print('Welcome!')
flag = input('your flag please:')
if l = len(flag) < 16:
    flag += '\x00' * (16 - l)
else:
    flag = flag[:16]
util.run('./prog', flag)
result = util.get_result()
if result:
    print('Correct!')
else:
    print('Wrong...')

```



flag 长度为 16





## IDA 查看



IDA 查看 **util.pyd** 文件  找到 `util.run`文件   sub_180028BF0 函数

