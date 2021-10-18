# [羊城杯 2020]login

## 初步观察

运行文件 通过图标可知该程序为 python代码 封装成`.exe`程序

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-04_14-32-29.png)

## 提取.pyc文件

该`.exe`文件说由 `Python`使用 `PYinstaller`打包的，使用对应脚本来提取该文件`.pyc`文件

[PyInstaller Extractor download | SourceForge.net](https://sourceforge.net/projects/pyinstallerextractor/)

>***\*py是源文件，\****
>
>***\*pyc是源文件编译后的文件\****
>
>***\*pyo是源文件优化编译后的文件\****
>
>***\*pyd是其他语言写的python库\****

**将下载后的 pyinstxtractor.py文件，与目标 attachment 文件放置于同一目录下，运行以下指令 **

```
 python pyinstxtractor.py attachment.exe
```

然后得到对应的文件夹，根据文件夹中文件名分析，可知`login.pyc`即为我们所需



## 反编译

借助在线网站[python反编译 - 在线工具 (tool.lu)](https://tool.lu/pyc/) 进行反编译

发现`login.pyc`反编译失败，尝试同目录下的`struct.pyc`反编译成功，因此使用`010editor`查看16进制，发现`login.pyc`魔术不对，对照`struct.pyc`进行修改，保存；即可反编译

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-09-04_15-22-00.png)



## python代码

```python
#!/usr/bin/env python
# visit https://tool.lu/pyc/ for more information
import sys
input1 = input('input something:')
if len(input1) != 14:
    print('Wrong length!')
    sys.exit()
code = []
for i in range(13):
    code.append(ord(input1[i]) ^ ord(input1[i + 1]))

code.append(ord(input1[13]))
a1 = code[2]
a2 = code[1]
a3 = code[0]
a4 = code[3]
a5 = code[4]
a6 = code[5]
a7 = code[6]
a8 = code[7]
a9 = code[9]
a10 = code[8]
a11 = code[10]
a12 = code[11]
a13 = code[12]
a14 = code[13]
if ((((a1 * 88 + a2 * 67 + a3 * 65 - a4 * 5) + a5 * 43 + a6 * 89 + a7 * 25 + a8 * 13 - a9 * 36) + a10 * 15 + a11 * 11 + a12 * 47 - a13 * 60) + a14 * 29 == 22748) & ((((a1 * 89 + a2 * 7 + a3 * 12 - a4 * 25) + a5 * 41 + a6 * 23 + a7 * 20 - a8 * 66) + a9 * 31 + a10 * 8 + a11 * 2 - a12 * 41 - a13 * 39) + a14 * 17 == 7258) & ((((a1 * 28 + a2 * 35 + a3 * 16 - a4 * 65) + a5 * 53 + a6 * 39 + a7 * 27 + a8 * 15 - a9 * 33) + a10 * 13 + a11 * 101 + a12 * 90 - a13 * 34) + a14 * 23 == 26190) & ((((a1 * 23 + a2 * 34 + a3 * 35 - a4 * 59) + a5 * 49 + a6 * 81 + a7 * 25 + (a8 << 7) - a9 * 32) + a10 * 75 + a11 * 81 + a12 * 47 - a13 * 60) + a14 * 29 == 37136) & (((a1 * 38 + a2 * 97 + a3 * 35 - a4 * 52) + a5 * 42 + a6 * 79 + a7 * 90 + a8 * 23 - a9 * 36) + a10 * 57 + a11 * 81 + a12 * 42 - a13 * 62 - a14 * 11 == 27915) & ((((a1 * 22 + a2 * 27 + a3 * 35 - a4 * 45) + a5 * 47 + a6 * 49 + a7 * 29 + a8 * 18 - a9 * 26) + a10 * 35 + a11 * 41 + a12 * 40 - a13 * 61) + a14 * 28 == 17298) & ((((a1 * 12 + a2 * 45 + a3 * 35 - a4 * 9 - a5 * 42) + a6 * 86 + a7 * 23 + a8 * 85 - a9 * 47) + a10 * 34 + a11 * 76 + a12 * 43 - a13 * 44) + a14 * 65 == 19875) & (((a1 * 79 + a2 * 62 + a3 * 35 - a4 * 85) + a5 * 33 + a6 * 79 + a7 * 86 + a8 * 14 - a9 * 30) + a10 * 25 + a11 * 11 + a12 * 57 - a13 * 50 - a14 * 9 == 22784) & ((((a1 * 8 + a2 * 6 + a3 * 64 - a4 * 85) + a5 * 73 + a6 * 29 + a7 * 2 + a8 * 23 - a9 * 36) + a10 * 5 + a11 * 2 + a12 * 47 - a13 * 64) + a14 * 27 == 9710) & (((((a1 * 67 - a2 * 68) + a3 * 68 - a4 * 51 - a5 * 43) + a6 * 81 + a7 * 22 - a8 * 12 - a9 * 38) + a10 * 75 + a11 * 41 + a12 * 27 - a13 * 52) + a14 * 31 == 13376) & ((((a1 * 85 + a2 * 63 + a3 * 5 - a4 * 51) + a5 * 44 + a6 * 36 + a7 * 28 + a8 * 15 - a9 * 6) + a10 * 45 + a11 * 31 + a12 * 7 - a13 * 67) + a14 * 78 == 24065) & ((((a1 * 47 + a2 * 64 + a3 * 66 - a4 * 5) + a5 * 43 + a6 * 112 + a7 * 25 + a8 * 13 - a9 * 35) + a10 * 95 + a11 * 21 + a12 * 43 - a13 * 61) + a14 * 20 == 27687) & (((a1 * 89 + a2 * 67 + a3 * 85 - a4 * 25) + a5 * 49 + a6 * 89 + a7 * 23 + a8 * 56 - a9 * 92) + a10 * 14 + a11 * 89 + a12 * 47 - a13 * 61 - a14 * 29 == 29250) & (((a1 * 95 + a2 * 34 + a3 * 62 - a4 * 9 - a5 * 43) + a6 * 83 + a7 * 25 + a8 * 12 - a9 * 36) + a10 * 16 + a11 * 51 + a12 * 47 - a13 * 60 - a14 * 24 == 15317):
    print('flag is GWHT{md5(your_input)}')
    print('Congratulations and have fun!')
else:
    print('Sorry,plz try again...')
```



## Z3求解器求解和与运算

### Z3求解器脚本

```python
from z3 import *

a1 = Int('a1')
a2 = Int('a2')
a3 = Int('a3')
a4 = Int('a4')
a5 = Int('a5')
a6 = Int('a6')
a7 = Int('a7')
a8 = Int('a8')
a9 = Int('a9')
a10 = Int('a10')
a11 = Int('a11')
a12 = Int('a12')
a13 = Int('a13')
a14 = Int('a14')

s = Solver()
s.add(a1*88 + a2*67 + a3*65 - a4*5 + a5*43 + a6 * 89 + a7*25+  a8*13- a9*36 + a10*15 + a11*11 + a12*47 - a13*60 + a14*29 == 22748)
s.add(a1*89 + a2*7 + a3*12 - a4*25 + a5*41 + a6 * 23 + a7*20 -  a8*66 + a9*31 +a10*8 + a11*2 - a12*41 - a13*39 + a14*17== 7258)
s.add(a1*28 + a2*35 + a3*16 - a4*65 + a5*53 + a6 * 39 + a7*27+  a8*15- a9*33 +a10*13 + a11*101 + a12*90 - a13*34 + a14*23 == 26190)
s.add(a1*23 + a2*34 + a3*35 - a4*59 + a5*49 + a6 * 81 + a7*25+  a8*128- a9*32 +a10*75 + a11*81 + a12*47 - a13*60 + a14*29== 37136)
s.add(a1*38 + a2*97 + a3*35 - a4*52 + a5*42 + a6 * 79 + a7*90+  a8*23- a9*36 +a10*57 + a11*81 + a12*42 - a13*62 - a14*11 == 27915)
s.add(a1*22 + a2*27 + a3*35 - a4*45 + a5*47 + a6 * 49 + a7*29+  a8*18- a9*26 +a10*35 + a11*41 + a12*40 - a13*61 + a14*28 == 17298)
s.add(a1*12 + a2*45 + a3*35 - a4*9 - a5*42 + a6 * 86 + a7*23+  a8*85- a9*47 +a10*34 + a11*76 + a12*43 - a13*44 + a14*65 == 19875)
s.add(a1*79 + a2*62 + a3*35 - a4*85 + a5*33 + a6 * 79 + a7*86+  a8*14- a9*30 +a10*25 + a11*11 + a12*57 - a13*50 - a14*9 == 22784)
s.add(a1*8 + a2*6 + a3*64 - a4*85 + a5*73 + a6 * 29 + a7*2+  a8*23- a9*36 +a10*5 + a11*2 + a12*47 - a13*64 + a14*27 == 9710)
s.add(a1*67 - a2*68 + a3*68 - a4*51 - a5*43 + a6 * 81 + a7*22-  a8*12- a9*38 +a10*75 + a11*41 + a12*27 - a13*52 + a14*31 == 13376)
s.add(a1*85 + a2*63 + a3*5 - a4*51 + a5*44 + a6 * 36 + a7*28+  a8*15- a9*6 +a10*45 + a11*31 + a12*7 - a13*67 + a14*78 == 24065)
s.add(a1*47 + a2*64 + a3*66 - a4*5 + a5*43 + a6 * 112 + a7*25+  a8*13- a9*35 +a10*95 + a11*21 + a12*43 - a13*61 + a14*20 == 27687)
s.add(a1*89 + a2*67 + a3*85 - a4*25 + a5*49 + a6 * 89 + a7*23+  a8*56- a9*92 +a10*14 + a11*89 + a12*47 - a13*61 - a14*29 == 29250)
s.add(a1*95 + a2*34 + a3*62 - a4*9 - a5*43 + a6 * 83 + a7*25+  a8*12- a9*36 +a10*16 + a11*51 + a12*47 - a13*60 - a14*24 == 15317)

if s.check()==sat:
    result = s.model()
print(result)
```



>[a2 = 24, 
> a13 = 88,
> a14 = 33,
> a6 = 43, 
> a9 = 52,
> a5 = 104,
> a12 = 74,
> a7 = 28,
> a1 = 119,
> a10 = 108,
> a11 = 88,
> a8 = 91,
> a4 = 7,
> a3 = 10]

### 与运算求解

```python
import hashlib
code = [10,24,119,7,104,43,28,91,108,52,88,74,88,33]
flag = ""

for i in range(13,0,-1):
    code[i-1] = code[i]^code[i-1]
    flag += chr(code[i-1])

flag = flag[::-1]
flag += chr(33)
#print(flag)
print(hashlib.md5(str(flag).encode()).hexdigest())
```

