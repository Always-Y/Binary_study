# Hyperthreading

---

- 花指令
- Z3求解器
- 反调试

---

### 运行程序查看特征：

![图1](D:\Binary_Study\2020CISCN\2.PNG)

如上图所示，直接要求输入flag，简单明了；



### 查壳

使用工具查壳，无壳，32位程序，简单介绍



### IDA 静态分析

具有大量得花指令，反调试不好操作；

> 程序一开始创建三个线程，第一个线程为关键线程，会对输入的数据做变换；第二个线程会判断是否处于调试状态，如果处于调试状态就会破坏数据；第三个线程是检测是否处于调试环境中，如果处于调试环境就退出程序。

注意 ，在第二个线程中，下面这个关键指令如下；具有ALSR保护，基址可能不同，RVA相同

```assembly
.text:00AC1200     sub_AC1200      proc near               ; DATA XREF: sub_AC1270+4B↓o
.text:00AC1200 000                 push    ebp
.text:00AC1201 004                 mov     ebp, esp
.text:00AC1203 004                 push    ecx
.text:00AC1204 008                 push    ebx
.text:00AC1205 00C                 push    esi
.text:00AC1206 010                 push    edi
.text:00AC1207 014                 mov     eax, large fs:30h ; PEB的位置
.text:00AC120D
.text:00AC120D     loc_AC120D:                             ; 花指令较多，吾爱破解OD自带去除花指令(能识别一部分)可以借鉴
.text:00AC120D 014                 nop
.text:00AC120E 014                 nop
.text:00AC120F 014                 nop                     ; Keypatch modified this from:
.text:00AC120F                                             ;   db 0C0h
.text:00AC1210 014                 dec     eax
.text:00AC1211 014                 movzx   eax, byte ptr [eax+2] ; IsbeingDebuged 标志，fs:[30]后0x2的偏移处，正常运行值为0，调试时为1
.text:00AC1215 014                 mov     [ebp-4], eax
.text:00AC1218 014                 movzx   ecx, byte ptr [ebp-4]
.text:00AC121C 014                 xor     eax, eax
.text:00AC121E 014                 imul    ebx, ecx, 64h   ; ebx = ecx *0x64h
.text:00AC1221
.text:00AC1221     loc_AC1221:                             ; CODE XREF: sub_AC1200+2D↓j
.text:00AC1221 014                 mov     dl, bl
.text:00AC1223 014                 add     byte_AC336C[eax], dl
.text:00AC1229 014                 inc     eax
.text:00AC122A 014                 cmp     eax, 2Ah
.text:00AC122D 014                 jl      short loc_AC1221
.text:00AC122F 014                 pop     edi
.text:00AC1230 010                 pop     esi
.text:00AC1231 00C                 xor     eax, eax
.text:00AC1233 00C                 pop     ebx
.text:00AC1234 008                 mov     esp, ebp
.text:00AC1236 004                 pop     ebp
.text:00AC1237 000                 retn    4
.text:00AC1237     sub_AC1200      endp
.text:00AC1237
```







### 脚本

**脚本一(借鉴)：**

```python
res=[0xDD, 0x5B, 0x9E, 0x1D, 0x20, 0x9E, 0x90, 0x91, 0x90, 0x90, 0x91, 0x92, 0xDE, 0x8B, 0x11, 0xD1, 0x1E, 0x9E, 0x8B, 0x51, 0x11, 0x50, 0x51, 0x8B, 0x9E, 0x5D, 0x5D, 0x11, 0x8B, 0x90, 0x12, 0x91, 0x50, 0x12, 0xD2, 0x91, 0x92, 0x1E, 0x9E, 0x90, 0xD2, 0x9F]
for i in range(42):
    res[i]=((res[i]-35)^0x23)&0xff #取8位
print(res)
for i in range(42):
    print(chr(((res[i]>>6)&0xff)^((res[i]<<2)&0xff)),end="")	#&0xff取8位，移动后可能自动扩充数据(类型)范围
```



**脚本二(自己的)：**

```python
# -*- coding:utf-8 -*-

from z3 import *

#v46 = Array()

v46,v47,v48,v49,v50,v51=BitVecs('v46 v47 v48 v49 v50 v51',8)
v68,v69,v70,v71,v72,v73,v74,v75,v76,v77,v78,v79,v80,v81,v82,v83,v84,v85,v86,v87 = BitVecs('v68 v69 v70 v71 v72 v73 v74 v75 v76 v77 v78 v79 v80 v81 v82 v83 v84 v85 v86 v87',8)
v61,v62,v63,v64,v65,v66,v67 = BitVecs('v61 v62 v63 v64 v65 v66 v67',8)
v55,v56,v57,v58,v59,v60 =BitVecs('v55 v56 v57 v58 v59 v60',8)
v52,v53,v54 =BitVecs('v52 v53 v54',8)

s =Solver()
s.add(((v46<<6)^(v46>>2)^0x23)+0x23==0xDD)
s.add(((v47<<6)^(v47>>2)^0x23)+0x23==0x5B)
s.add(((v48<<6)^(v48>>2)^0x23)+0x23==0x9e)
s.add(((v49<<6)^(v49>>2)^0x23)+0x23==0x1d)
s.add(((v50<<6)^(v50>>2)^0x23)+0x23==0x20)
s.add(((v51<<6)^(v51>>2)^0x23)+0x23==0x9e)
s.add(((v52<<6)^(v52>>2)^0x23)+0x23==0x90)
s.add(((v53<<6)^(v53>>2)^0x23)+0x23==0x91)
s.add(((v54<<6)^(v54>>2)^0x23)+0x23==0x90)
s.add(((v55<<6)^(v55>>2)^0x23)+0x23==0x90)
s.add(((v56<<6)^(v56>>2)^0x23)+0x23==0x91)
s.add(((v57<<6)^(v57>>2)^0x23)+0x23==0x92)
s.add(((v58<<6)^(v58>>2)^0x23)+0x23==0xde)
s.add(((v59<<6)^(v59>>2)^0x23)+0x23==0x8b)
s.add(((v60<<6)^(v60>>2)^0x23)+0x23==0x11)
s.add(((v61<<6)^(v61>>2)^0x23)+0x23==0xd1)
s.add(((v62<<6)^(v62>>2)^0x23)+0x23==0x1e)
s.add(((v63<<6)^(v63>>2)^0x23)+0x23==0x9e)
s.add(((v64<<6)^(v64>>2)^0x23)+0x23==0x8b)
s.add(((v65<<6)^(v65>>2)^0x23)+0x23==0x51)
s.add(((v66<<6)^(v66>>2)^0x23)+0x23==0x11)
s.add(((v67<<6)^(v67>>2)^0x23)+0x23==0x50)
s.add(((v68<<6)^(v68>>2)^0x23)+0x23==0x51)
s.add(((v69<<6)^(v69>>2)^0x23)+0x23==0x8b)
s.add(((v70<<6)^(v70>>2)^0x23)+0x23==0x9e)
s.add(((v71<<6)^(v71>>2)^0x23)+0x23==0x5d)
s.add(((v72<<6)^(v72>>2)^0x23)+0x23==0x5d)
s.add(((v73<<6)^(v73>>2)^0x23)+0x23==0x11)
s.add(((v74<<6)^(v74>>2)^0x23)+0x23==0x8b)
s.add(((v75<<6)^(v75>>2)^0x23)+0x23==0x90)
s.add(((v76<<6)^(v76>>2)^0x23)+0x23==0x12)
s.add(((v77<<6)^(v77>>2)^0x23)+0x23==0x91)
s.add(((v78<<6)^(v78>>2)^0x23)+0x23==0x50)
s.add(((v79<<6)^(v79>>2)^0x23)+0x23==0x12)
s.add(((v80<<6)^(v80>>2)^0x23)+0x23==0xd2)
s.add(((v81<<6)^(v81>>2)^0x23)+0x23==0x91)
s.add(((v82<<6)^(v82>>2)^0x23)+0x23==0x92)
s.add(((v83<<6)^(v83>>2)^0x23)+0x23==0x1e)
s.add(((v84<<6)^(v84>>2)^0x23)+0x23==0x9e)
s.add(((v85<<6)^(v85>>2)^0x23)+0x23==0x90)
s.add(((v86<<6)^(v86>>2)^0x23)+0x23==0xd2)
s.add(((v87<<6)^(v87>>2)^0x23)+0x23==0x9f)

if s.check()==sat:
    m = s.model()
#print(chr(m[v46].as_long()))
for i in range(46,88):
    n = BitVec('v'+str(i),8)            #构建的字符串要转化为BitVec()类型，才能用于后面索引
    print(chr(m[n].as_long())),         #后面加一个逗号可以使打印不换行


```





### FLAG

flag{a959951b-76ca-4784-add7-93583251ca92}