# [GXYCTF2019]simple CPP

---

- Z3求解器的使用
- 算法有特点，分析

---

## 运行程序查看程序特征

如下图所示，运行程序后，让我们直接输入flag:

![图1](D:\Binary_Study\[GXYCTF2019]simple CPP\1.PNG)





## 查壳

![图2](D:\Binary_Study\[GXYCTF2019]simple CPP\2.png)

由工具可知，该程序无壳，且为C/C++编写的64位程序



## IDA 静态分析

定位到关键位置，F5得到反汇编代码，如下所示

```c
__int64 sub_7FF66C231290()
{
  bool v0; // si
  _QWORD *v1; // rax
  unsigned __int8 *v2; // rax
  unsigned __int8 *v3; // rbx
  int v4; // er10
  __int64 v5; // r11
  _BYTE *v6; // r9
  void **v7; // r8
  __int64 v8; // rdi
  __int64 v9; // r15
  __int64 v10; // r12
  __int64 v11; // rbp
  signed int v12; // ecx
  unsigned __int8 *v13; // rdx
  __int64 v14; // rdi
  __int64 *v15; // r14
  __int64 y; // rbp
  __int64 x; // r13
  _QWORD *v18; // rdi
  __int64 v19; // r12
  __int64 v20; // r15
  __int64 v21; // rbp
  __int64 v22; // rdx
  __int64 v23; // rbp
  __int64 v24; // rbp
  __int64 v25; // r10
  __int64 v26; // rdi
  __int64 v27; // r8
  bool v28; // dl
  _QWORD *v29; // rax
  void *v30; // rdx
  _QWORD *v31; // rax
  _QWORD *v32; // rax
  _BYTE *v33; // rcx
  __int64 z; // [rsp+20h] [rbp-68h]
  void *Memory; // [rsp+30h] [rbp-58h]
  unsigned __int64 v37; // [rsp+40h] [rbp-48h]
  unsigned __int64 v38; // [rsp+48h] [rbp-40h]

  v0 = 0;
  v37 = 0i64;
  v38 = 15i64;
  LOBYTE(Memory) = 0;
  v1 = sub_7FF66C2319C0(std::cout, (__int64)"I'm a first timer of Logic algebra , how about you?");
  std::basic_ostream<char,std::char_traits<char>>::operator<<(v1, sub_7FF66C231B90);
  sub_7FF66C2319C0(std::cout, (__int64)"Let's start our game,Please input your flag:");
  sub_7FF66C231DE0(std::cin, &Memory);          // 输入
  std::basic_ostream<char,std::char_traits<char>>::operator<<(std::cout, sub_7FF66C231B90);
  if ( v37 - 5 > 0x19 )                         // v37为输入字符串长度 + \n  GXY{24个字符}
  {
    v32 = sub_7FF66C2319C0(std::cout, (__int64)"Wrong input ,no GXY{} in input words");
    std::basic_ostream<char,std::char_traits<char>>::operator<<(v32, sub_7FF66C231B90);
    goto LABEL_45;
  }
  v2 = (unsigned __int8 *)sub_7FF66C2324C8(0x20ui64);
  v3 = v2;
  if ( v2 )
  {
    *(_QWORD *)v2 = 0i64;
    *((_QWORD *)v2 + 1) = 0i64;
    *((_QWORD *)v2 + 2) = 0i64;
    *((_QWORD *)v2 + 3) = 0i64;
  }
  else
  {
    v3 = 0i64;
  }
  v4 = 0;
  if ( v37 > 0 )
  {
    v5 = 0i64;
    do
    {
      v6 = &Memory;
      if ( v38 >= 0x10 )
        v6 = Memory;                            // 不执行
      v7 = &Dst;
      if ( (unsigned __int64)qword_7FF66C236060 >= 0x10 )
        v7 = (void **)Dst;                      // V7 =i_will_check_is_debug_or_not
      v3[v5] = v6[v5] ^ *((_BYTE *)v7 + v4++ % 27);// 对输入做了重要运算
      ++v5;
    }
    while ( v4 < v37 );
  }
  v8 = 0i64;
  v9 = 0i64;
  v10 = 0i64;
  v11 = 0i64;
  if ( (signed int)v37 > 30 )
    goto LABEL_28;
  v12 = 0;
  if ( (signed int)v37 <= 0 )
    goto LABEL_28;
  v13 = v3;                                     // 对上述处理得到的V3赋值给V13进一步处理
  do
  {
    v14 = *v13 + v8;                            // 重要循环
    ++v12;
    ++v13;
    switch ( v12 )
    {                                           // 将字符堆叠，例如 原本的是 db 2E 07 变为0x2E07
      case 8:
        v11 = v14;
        goto LABEL_24;
      case 16:                                  // 30/8=3...6
        v10 = v14;
        goto LABEL_24;
      case 24:
        v9 = v14;
LABEL_24:
        v14 = 0i64;
        break;
      case 32:
        sub_7FF66C2319C0(std::cout, (__int64)"ERRO,out of range");
        exit(1);
        break;
    }
    v8 = v14 << 8;
  }
  while ( v12 < (signed int)v37 );
  if ( v11 )
  {
    v15 = (__int64 *)sub_7FF66C2324C8(0x20ui64);
    *v15 = v11;
    v15[1] = v10;
    v15[2] = v9;                                // 以v15为首地址，依次对应堆叠得到的4个字符
    v15[3] = v8;
    goto LABEL_29;
  }
LABEL_28:
  v15 = 0i64;
LABEL_29:
  z = v15[2];
  y = v15[1];
  x = *v15;
  v18 = sub_7FF66C23223C(0x20ui64);
  if ( IsDebuggerPresent() )                    // 反调试
  {
    sub_7FF66C2319C0(std::cout, (__int64)"Hi , DO not debug me !");
    Sleep(0x7D0u);
    exit(0);
  }
  v19 = y & x;
  *v18 = y & x;
  v20 = z & ~x;
  v18[1] = v20;
  v21 = ~y;
  v22 = z & v21;
  v18[2] = z & v21;
  v23 = x & v21;
  v18[3] = v23;
  if ( v20 != 1176889593874i64 )                // z & ~x ==v20
  {
    v18[1] = 0i64;
    v20 = 0i64;
  }
  v24 = v20 | v19 | v22 | v23;
  v25 = v15[1];
  v26 = v15[2];
  v27 = v22 & *v15 | v26 & (v19 | v25 & ~*v15 | ~(v25 | *v15));
  v28 = 0;
  if ( v27 == 577031497978884115i64 )
    v28 = v24 == 4483974544037412639i64;
  if ( (v24 ^ v15[3]) == 4483974543195470111i64 )// w =4483974543195470111^4483974544037412639
    v0 = v28;
  if ( (v20 | v19 | v25 & v26) != (~*v15 & v26 | 0xC00020130082C0Ci64) || v0 != 1 )
  {
    sub_7FF66C2319C0(std::cout, (__int64)"Wrong answer!try again");
    j_j_free(v3);
  }
  else
  {
    v29 = sub_7FF66C2319C0(std::cout, (__int64)"Congratulations!flag is GXY{");
    v30 = &Memory;
    if ( v38 >= 0x10 )
      v30 = Memory;
    v31 = sub_7FF66C231FD0(v29, (__int64)v30, v37);
    sub_7FF66C2319C0(v31, (__int64)"}");
    j_j_free(v3);
  }
LABEL_45:
  if ( v38 >= 0x10 )
  {
    v33 = Memory;
    if ( v38 + 1 >= 0x1000 )
    {
      v33 = (_BYTE *)*((_QWORD *)Memory - 1);
      if ( (unsigned __int64)((_BYTE *)Memory - v33 - 8) > 0x1F )
        invalid_parameter_noinfo_noreturn();
    }
    j_j_free(v33);
  }
  return 0i64;
}
```



整个流程是，输入的字符串与特定字符串逐个字符与运算，然后将得到的数据堆叠，在进行运算比较，你想求解血，需要先使用Z3求解器

得到与运算后结果。



## Z3求解

```py
# -*- coding:utf-8 -*-

from z3 import *

x,y,z,w=BitVecs('x y z w',64)       #需要进行位运算，使用BitVecs()类型

s=Solver()          #创建求解器

s.add((z&(~x))==1176889593874)
s.add((4483974544037412639^w)==4483974543195470111)
s.add(((z&(~y))&x|z&((y&x)|y&~x|~(y|x)))==577031497978884115)
s.add(((z & (~x))|(y & x)|(z&(~y))|(x&~y))==4483974544037412639)
s.add(((z&(~x))|(y&x)|y&z) == ((~x) &z|0xC00020130082C0C))


print s.check()
m = s.model()
for i in m:
    print("%s = 0x%x"%(i,m[i].as_long()))       #得到的结果为字符串形式，需要转换后，格式化输出
```



## 求解flag

