# 2020CISCN-----Z3

---

- Z3求解器数组的使用
- IDA python的使用

---

### 运行程序查看特征

![图1](D:\Binary_Study\2020CISCN\1.png)

如上图所示，直接要求输入flag，简单命令



### 查壳

使用工具查壳，无壳，64位程序；

### IDA静态分析

根据字符串的交叉引用定位到关键位置，F5反编译得到如下代码：

```c
// local variable allocation has failed, the output may be wrong!
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [rsp+20h] [rbp-60h]
  int v5; // [rsp+24h] [rbp-5Ch]
  int v6; // [rsp+28h] [rbp-58h]
  int v7; // [rsp+2Ch] [rbp-54h]
  int v8; // [rsp+30h] [rbp-50h]
  int v9; // [rsp+34h] [rbp-4Ch]
  int v10; // [rsp+38h] [rbp-48h]
  int v11; // [rsp+3Ch] [rbp-44h]
  int v12; // [rsp+40h] [rbp-40h]
  int v13; // [rsp+44h] [rbp-3Ch]
  int v14; // [rsp+48h] [rbp-38h]
  int v15; // [rsp+4Ch] [rbp-34h]
  int v16; // [rsp+50h] [rbp-30h]
  int v17; // [rsp+54h] [rbp-2Ch]
  int v18; // [rsp+58h] [rbp-28h]
  int v19; // [rsp+5Ch] [rbp-24h]
  int v20; // [rsp+60h] [rbp-20h]
  int v21; // [rsp+64h] [rbp-1Ch]
  int v22; // [rsp+68h] [rbp-18h]
  int v23; // [rsp+6Ch] [rbp-14h]
  int v24; // [rsp+70h] [rbp-10h]
  int v25; // [rsp+74h] [rbp-Ch]
  int v26; // [rsp+78h] [rbp-8h]
  int v27; // [rsp+7Ch] [rbp-4h]
  int v28; // [rsp+80h] [rbp+0h]
  int v29; // [rsp+84h] [rbp+4h]
  int v30; // [rsp+88h] [rbp+8h]
  int v31; // [rsp+8Ch] [rbp+Ch]
  int v32; // [rsp+90h] [rbp+10h]
  int v33; // [rsp+94h] [rbp+14h]
  int v34; // [rsp+98h] [rbp+18h]
  int v35; // [rsp+9Ch] [rbp+1Ch]
  int v36; // [rsp+A0h] [rbp+20h]
  int v37; // [rsp+A4h] [rbp+24h]
  int v38; // [rsp+A8h] [rbp+28h]
  int v39; // [rsp+ACh] [rbp+2Ch]
  int v40; // [rsp+B0h] [rbp+30h]
  int v41; // [rsp+B4h] [rbp+34h]
  int v42; // [rsp+B8h] [rbp+38h]
  int v43; // [rsp+BCh] [rbp+3Ch]
  int v44; // [rsp+C0h] [rbp+40h]
  int v45; // [rsp+C4h] [rbp+44h]
  unsigned __int8 v46; // [rsp+D0h] [rbp+50h]
  unsigned __int8 v47; // [rsp+D1h] [rbp+51h]
  unsigned __int8 v48; // [rsp+D2h] [rbp+52h]
  unsigned __int8 v49; // [rsp+D3h] [rbp+53h]
  unsigned __int8 v50; // [rsp+D4h] [rbp+54h]
  unsigned __int8 v51; // [rsp+D5h] [rbp+55h]
  unsigned __int8 v52; // [rsp+D6h] [rbp+56h]
  unsigned __int8 v53; // [rsp+D7h] [rbp+57h]
  unsigned __int8 v54; // [rsp+D8h] [rbp+58h]
  unsigned __int8 v55; // [rsp+D9h] [rbp+59h]
  unsigned __int8 v56; // [rsp+DAh] [rbp+5Ah]
  unsigned __int8 v57; // [rsp+DBh] [rbp+5Bh]
  unsigned __int8 v58; // [rsp+DCh] [rbp+5Ch]
  unsigned __int8 v59; // [rsp+DDh] [rbp+5Dh]
  unsigned __int8 v60; // [rsp+DEh] [rbp+5Eh]
  unsigned __int8 v61; // [rsp+DFh] [rbp+5Fh]
  unsigned __int8 v62; // [rsp+E0h] [rbp+60h]
  unsigned __int8 v63; // [rsp+E1h] [rbp+61h]
  unsigned __int8 v64; // [rsp+E2h] [rbp+62h]
  unsigned __int8 v65; // [rsp+E3h] [rbp+63h]
  unsigned __int8 v66; // [rsp+E4h] [rbp+64h]
  unsigned __int8 v67; // [rsp+E5h] [rbp+65h]
  unsigned __int8 v68; // [rsp+E6h] [rbp+66h]
  unsigned __int8 v69; // [rsp+E7h] [rbp+67h]
  unsigned __int8 v70; // [rsp+E8h] [rbp+68h]
  unsigned __int8 v71; // [rsp+E9h] [rbp+69h]
  unsigned __int8 v72; // [rsp+EAh] [rbp+6Ah]
  unsigned __int8 v73; // [rsp+EBh] [rbp+6Bh]
  unsigned __int8 v74; // [rsp+ECh] [rbp+6Ch]
  unsigned __int8 v75; // [rsp+EDh] [rbp+6Dh]
  unsigned __int8 v76; // [rsp+EEh] [rbp+6Eh]
  unsigned __int8 v77; // [rsp+EFh] [rbp+6Fh]
  unsigned __int8 v78; // [rsp+F0h] [rbp+70h]
  unsigned __int8 v79; // [rsp+F1h] [rbp+71h]
  unsigned __int8 v80; // [rsp+F2h] [rbp+72h]
  unsigned __int8 v81; // [rsp+F3h] [rbp+73h]
  unsigned __int8 v82; // [rsp+F4h] [rbp+74h]
  unsigned __int8 v83; // [rsp+F5h] [rbp+75h]
  unsigned __int8 v84; // [rsp+F6h] [rbp+76h]
  unsigned __int8 v85; // [rsp+F7h] [rbp+77h]
  unsigned __int8 v86; // [rsp+F8h] [rbp+78h]
  unsigned __int8 v87; // [rsp+F9h] [rbp+79h]
  int Dst[43]; // [rsp+110h] [rbp+90h]
  int i; // [rsp+1BCh] [rbp+13Ch]

  _main(*(_QWORD *)&argc, argv, envp);
  memcpy(Dst, &dword_404020, 0xA8ui64);
  printf("plz input your flag:");
  scanf("%42s", &v46);
  v4 = 34 * v49 + 12 * v46 + 53 * v47 + 6 * v48 + 58 * v50 + 36 * v51 + v52;
  v5 = 27 * v50 + 73 * v49 + 12 * v48 + 83 * v46 + 85 * v47 + 96 * v51 + 52 * v52;
  v6 = 24 * v48 + 78 * v46 + 53 * v47 + 36 * v49 + 86 * v50 + 25 * v51 + 46 * v52;
  v7 = 78 * v47 + 39 * v46 + 52 * v48 + 9 * v49 + 62 * v50 + 37 * v51 + 84 * v52;
  v8 = 48 * v50 + 14 * v48 + 23 * v46 + 6 * v47 + 74 * v49 + 12 * v51 + 83 * v52;
  v9 = 15 * v51 + 48 * v50 + 92 * v48 + 85 * v47 + 27 * v46 + 42 * v49 + 72 * v52;
  v10 = 26 * v51 + 67 * v49 + 6 * v47 + 4 * v46 + 3 * v48 + 68 * v52;
  v11 = 34 * v56 + 12 * v53 + 53 * v54 + 6 * v55 + 58 * v57 + 36 * v58 + v59;
  v12 = 27 * v57 + 73 * v56 + 12 * v55 + 83 * v53 + 85 * v54 + 96 * v58 + 52 * v59;
  v13 = 24 * v55 + 78 * v53 + 53 * v54 + 36 * v56 + 86 * v57 + 25 * v58 + 46 * v59;
  v14 = 78 * v54 + 39 * v53 + 52 * v55 + 9 * v56 + 62 * v57 + 37 * v58 + 84 * v59;
  v15 = 48 * v57 + 14 * v55 + 23 * v53 + 6 * v54 + 74 * v56 + 12 * v58 + 83 * v59;
  v16 = 15 * v58 + 48 * v57 + 92 * v55 + 85 * v54 + 27 * v53 + 42 * v56 + 72 * v59;
  v17 = 26 * v58 + 67 * v56 + 6 * v54 + 4 * v53 + 3 * v55 + 68 * v59;
  v18 = 34 * v63 + 12 * v60 + 53 * v61 + 6 * v62 + 58 * v64 + 36 * v65 + v66;
  v19 = 27 * v64 + 73 * v63 + 12 * v62 + 83 * v60 + 85 * v61 + 96 * v65 + 52 * v66;
  v20 = 24 * v62 + 78 * v60 + 53 * v61 + 36 * v63 + 86 * v64 + 25 * v65 + 46 * v66;
  v21 = 78 * v61 + 39 * v60 + 52 * v62 + 9 * v63 + 62 * v64 + 37 * v65 + 84 * v66;
  v22 = 48 * v64 + 14 * v62 + 23 * v60 + 6 * v61 + 74 * v63 + 12 * v65 + 83 * v66;
  v23 = 15 * v65 + 48 * v64 + 92 * v62 + 85 * v61 + 27 * v60 + 42 * v63 + 72 * v66;
  v24 = 26 * v65 + 67 * v63 + 6 * v61 + 4 * v60 + 3 * v62 + 68 * v66;
  v25 = 34 * v70 + 12 * v67 + 53 * v68 + 6 * v69 + 58 * v71 + 36 * v72 + v73;
  v26 = 27 * v71 + 73 * v70 + 12 * v69 + 83 * v67 + 85 * v68 + 96 * v72 + 52 * v73;
  v27 = 24 * v69 + 78 * v67 + 53 * v68 + 36 * v70 + 86 * v71 + 25 * v72 + 46 * v73;
  v28 = 78 * v68 + 39 * v67 + 52 * v69 + 9 * v70 + 62 * v71 + 37 * v72 + 84 * v73;
  v29 = 48 * v71 + 14 * v69 + 23 * v67 + 6 * v68 + 74 * v70 + 12 * v72 + 83 * v73;
  v30 = 15 * v72 + 48 * v71 + 92 * v69 + 85 * v68 + 27 * v67 + 42 * v70 + 72 * v73;
  v31 = 26 * v72 + 67 * v70 + 6 * v68 + 4 * v67 + 3 * v69 + 68 * v73;
  v32 = 34 * v77 + 12 * v74 + 53 * v75 + 6 * v76 + 58 * v78 + 36 * v79 + v80;
  v33 = 27 * v78 + 73 * v77 + 12 * v76 + 83 * v74 + 85 * v75 + 96 * v79 + 52 * v80;
  v34 = 24 * v76 + 78 * v74 + 53 * v75 + 36 * v77 + 86 * v78 + 25 * v79 + 46 * v80;
  v35 = 78 * v75 + 39 * v74 + 52 * v76 + 9 * v77 + 62 * v78 + 37 * v79 + 84 * v80;
  v36 = 48 * v78 + 14 * v76 + 23 * v74 + 6 * v75 + 74 * v77 + 12 * v79 + 83 * v80;
  v37 = 15 * v79 + 48 * v78 + 92 * v76 + 85 * v75 + 27 * v74 + 42 * v77 + 72 * v80;
  v38 = 26 * v79 + 67 * v77 + 6 * v75 + 4 * v74 + 3 * v76 + 68 * v80;
  v39 = 34 * v84 + 12 * v81 + 53 * v82 + 6 * v83 + 58 * v85 + 36 * v86 + v87;
  v40 = 27 * v85 + 73 * v84 + 12 * v83 + 83 * v81 + 85 * v82 + 96 * v86 + 52 * v87;
  v41 = 24 * v83 + 78 * v81 + 53 * v82 + 36 * v84 + 86 * v85 + 25 * v86 + 46 * v87;
  v42 = 78 * v82 + 39 * v81 + 52 * v83 + 9 * v84 + 62 * v85 + 37 * v86 + 84 * v87;
  v43 = 48 * v85 + 14 * v83 + 23 * v81 + 6 * v82 + 74 * v84 + 12 * v86 + 83 * v87;
  v44 = 15 * v86 + 48 * v85 + 92 * v83 + 85 * v82 + 27 * v81 + 42 * v84 + 72 * v87;
  v45 = 26 * v86 + 67 * v84 + 6 * v82 + 4 * v81 + 3 * v83 + 68 * v87;
  for ( i = 0; i <= 41; ++i )
  {
    if ( *(&v4 + i) != Dst[i] )
    {
      printf("error");
      exit(0);
    }
  }
  printf("win");
  return 0;
}
```

过程简单，很明显输入的每个字符经过系列复杂的运算后与Dst数组逐一比较，相同这位flag，否者输入错误，而Dst可知是由

dword_404020复制而来，该数组的数据存储于程序中；所以整个过程主要是Z3求解



### Z3求解

- 使用IDApython 将Dst的值打印出来

**for i in range(0x30):print(str(hex(Dword(0x404020+4*i)))+',')**   ；0x404020即为首个数据开始的地址

- 调整代码，将原本多个变量合并定义为数组  方法：左键单击第一变量，进入该变量的地址，即数组的首地址，选择合并的范围，右键选择定义为数组；
- Z3求解

**第一种写法 变量通过重复的定义实现 **

```py
# -*- coding:utf-8 -*-

from z3 import *

cyphered = [0x4f17, 0x9cf6, 0x8ddb, 0x8ea6, 0x6929, 0x9911, 0x40a2, 0x2f3e, 0x62b6, 0x4b82, 0x486c, 0x4002,
0x52d7,0x2def, 0x28dc, 0x640d, 0x528f, 0x613b, 0x4781, 0x6b17, 0x3237, 0x2a93, 0x615f, 0x50be, 0x598e,
0x4656,0x5b31, 0x313a, 0x3010, 0x67fe, 0x4d5f, 0x58db, 0x3799, 0x60a0, 0x2750, 0x3759,0x8953, 0x7122, 0x81f9, 0x5524, 0x8971, 0x3a1d]

#定义Z3变量
Input = [0]*42
for i in range(42):
    Input[i] = Int('Input['+str(i)+']')     #Int()的参数需要为字符串类型

s = Solver()  # 构造求解器
s.add(cyphered[0] == (34 * Input[3]+ 12 * Input[0]+ 53 * Input[1]+ 6 * Input[2]+ 58 * Input[4]+ 36 * Input[5]+ Input[6]))
s.add(cyphered[1] == ( 27 * Input[4]+ 73 * Input[3]+ 12 * Input[2]+ 83 * Input[0]+ 85 * Input[1] + 96 * Input[5]+ 52 * Input[6]))
s.add(cyphered[2] == (24 * Input[2]+ 78 * Input[0]+ 53 * Input[1]+ 36 * Input[3] + 86 * Input[4]+ 25 * Input[5]+ 46 * Input[6]))
s.add(cyphered[3]== (78 * Input[1]+ 39 * Input[0]+ 52 * Input[2]+ 9 * Input[3]+ 62 * Input[4]+ 37 * Input[5]+ 84 * Input[6]))
.....
```



**第二种写法：使用Z3数组定义**

```py
# -*- coding:utf-8 -*-

from z3 import *

v4 = [0x4f17, 0x9cf6, 0x8ddb, 0x8ea6, 0x6929, 0x9911, 0x40a2, 0x2f3e, 0x62b6, 0x4b82, 0x486c, 0x4002,
0x52d7,0x2def, 0x28dc, 0x640d, 0x528f, 0x613b, 0x4781, 0x6b17, 0x3237, 0x2a93, 0x615f, 0x50be, 0x598e,
0x4656,0x5b31, 0x313a, 0x3010, 0x67fe, 0x4d5f, 0x58db, 0x3799, 0x60a0, 0x2750, 0x3759,0x8953, 0x7122, 0x81f9, 0x5524, 0x8971, 0x3a1d]

v5 = IntVector('x', 42)		#使用Z3数组定义

s = Solver()

s.add(v4[0]==34*v5[3]+12*v5[0]+53*v5[1]+6*v5[2]+58*v5[4]+36*v5[5]+v5[6])
s.add(v4[1]==27*v5[4]+73*v5[3]+12*v5[2]+83*v5[0]+85*v5[1]+96*v5[5]+52*v5[6])
s.add(v4[2]==24*v5[2]+78*v5[0]+53*v5[1]+36*v5[3]+86*v5[4]+25*v5[5]+46*v5[6])
s.add(v4[3]==78*v5[1]+39*v5[0]+52*v5[2]+9*v5[3]+62*v5[4]+37*v5[5]+84*v5[6])
s.add(v4[4]==48*v5[4]+14*v5[2]+23*v5[0]+6*v5[1]+74*v5[3]+12*v5[5]+83*v5[6])
s.add(v4[5]==15*v5[5]+48*v5[4]+92*v5[2]+85*v5[1]+27*v5[0]+42*v5[3]+72*v5[6])
s.add(v4[6]==26*v5[5]+67*v5[3]+6*v5[1]+4*v5[0]+3*v5[2]+68*v5[6])
s.add(v4[7]==34*v5[10]+12*v5[7]+53*v5[8]+6*v5[9]+58*v5[11]+36*v5[12]+v5[13])
s.add(v4[8]==27*v5[11]+73*v5[10]+12*v5[9]+83*v5[7]+85*v5[8]+96*v5[12]+52*v5[13])
s.add(v4[9]==24*v5[9]+78*v5[7]+53*v5[8]+36*v5[10]+86*v5[11]+25*v5[12]+46*v5[13])
s.add(v4[10]==78*v5[8]+39*v5[7]+52*v5[9]+9*v5[10]+62*v5[11]+37*v5[12]+84*v5[13])
s.add(v4[11]==48*v5[11]+14*v5[9]+23*v5[7]+6*v5[8]+74*v5[10]+12*v5[12]+83*v5[13])
s.add(v4[12]==15*v5[12]+48*v5[11]+92*v5[9]+85*v5[8]+27*v5[7]+42*v5[10]+72*v5[13])
s.add(v4[13]==26*v5[12]+67*v5[10]+6*v5[8]+4*v5[7]+3*v5[9]+68*v5[13])
s.add(v4[14]==34*v5[17]+12*v5[14]+53*v5[15]+6*v5[16]+58*v5[18]+36*v5[19]+v5[20])
s.add(v4[15]==27*v5[18]+73*v5[17]+12*v5[16]+83*v5[14]+85*v5[15]+96*v5[19]+52*v5[20])
s.add(v4[16]==24*v5[16]+78*v5[14]+53*v5[15]+36*v5[17]+86*v5[18]+25*v5[19]+46*v5[20])
s.add(v4[17]==78*v5[15]+39*v5[14]+52*v5[16]+9*v5[17]+62*v5[18]+37*v5[19]+84*v5[20])
s.add(v4[18]==48*v5[18]+14*v5[16]+23*v5[14]+6*v5[15]+74*v5[17]+12*v5[19]+83*v5[20])
s.add(v4[19]==15*v5[19]+48*v5[18]+92*v5[16]+85*v5[15]+27*v5[14]+42*v5[17]+72*v5[20])
s.add(v4[20]==26*v5[19]+67*v5[17]+6*v5[15]+4*v5[14]+3*v5[16]+68*v5[20])
s.add(v4[21]==34*v5[24]+12*v5[21]+53*v5[22]+6*v5[23]+58*v5[25]+36*v5[26]+v5[27])
s.add(v4[22]==27*v5[25]+73*v5[24]+12*v5[23]+83*v5[21]+85*v5[22]+96*v5[26]+52*v5[27])
s.add(v4[23]==24*v5[23]+78*v5[21]+53*v5[22]+36*v5[24]+86*v5[25]+25*v5[26]+46*v5[27])
s.add(v4[24]==78*v5[22]+39*v5[21]+52*v5[23]+9*v5[24]+62*v5[25]+37*v5[26]+84*v5[27])
s.add(v4[25]==48*v5[25]+14*v5[23]+23*v5[21]+6*v5[22]+74*v5[24]+12*v5[26]+83*v5[27])
s.add(v4[26]==15*v5[26]+48*v5[25]+92*v5[23]+85*v5[22]+27*v5[21]+42*v5[24]+72*v5[27])
s.add(v4[27]==26*v5[26]+67*v5[24]+6*v5[22]+4*v5[21]+3*v5[23]+68*v5[27])
s.add(v4[28]==34*v5[31]+12*v5[28]+53*v5[29]+6*v5[30]+58*v5[32]+36*v5[33]+v5[34])
s.add(v4[29]==27*v5[32]+73*v5[31]+12*v5[30]+83*v5[28]+85*v5[29]+96*v5[33]+52*v5[34])
s.add(v4[30]==24*v5[30]+78*v5[28]+53*v5[29]+36*v5[31]+86*v5[32]+25*v5[33]+46*v5[34])
s.add(v4[31]==78*v5[29]+39*v5[28]+52*v5[30]+9*v5[31]+62*v5[32]+37*v5[33]+84*v5[34])
s.add(v4[32]==48*v5[32]+14*v5[30]+23*v5[28]+6*v5[29]+74*v5[31]+12*v5[33]+83*v5[34])
s.add(v4[33]==15*v5[33]+48*v5[32]+92*v5[30]+85*v5[29]+27*v5[28]+42*v5[31]+72*v5[34])
s.add(v4[34]==26*v5[33]+67*v5[31]+6*v5[29]+4*v5[28]+3*v5[30]+68*v5[34])
s.add(v4[35]==34*v5[38]+12*v5[35]+53*v5[36]+6*v5[37]+58*v5[39]+36*v5[40]+v5[41])
s.add(v4[36]==27*v5[39]+73*v5[38]+12*v5[37]+83*v5[35]+85*v5[36]+96*v5[40]+52*v5[41])
s.add(v4[37]==24*v5[37]+78*v5[35]+53*v5[36]+36*v5[38]+86*v5[39]+25*v5[40]+46*v5[41])
s.add(v4[38]==78*v5[36]+39*v5[35]+52*v5[37]+9*v5[38]+62*v5[39]+37*v5[40]+84*v5[41])
s.add(v4[39]==48*v5[39]+14*v5[37]+23*v5[35]+6*v5[36]+74*v5[38]+12*v5[40]+83*v5[41])
s.add(v4[40]==15*v5[40]+48*v5[39]+92*v5[37]+85*v5[36]+27*v5[35]+42*v5[38]+72*v5[41])
s.add(v4[41]==26*v5[40]+67*v5[38]+6*v5[36]+4*v5[35]+3*v5[37]+68*v5[41])

if s.check() == sat:
    m = s.model()  #m是一个列表，形如[x__0 = 102,x__26 = 48,x__33 = 99,，，，，]
    res = ""
    for i in range(42):
        res += chr(m[v5[i]].as_long())		#数字字符先转化为数字，再转为字符

    print(res)
```



### FLAG

flag{7e171d43-63b9-4e18-990e-6e14c2afe648}