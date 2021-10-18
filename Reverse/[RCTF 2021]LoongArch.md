# LoongArch

## 原始汇编

给了一个 龙芯 的汇编代码 需要查阅指令手册 来进行分析

```
ld.d t0,sp,0 
ld.d t1,sp,8 
ld.d t2,sp,16 
ld.d t3,sp,24
ld.d t4,sp,32
ld.d t5,sp,40
ld.d t6,sp,48
ld.d t7,sp,56

xor t0,t0,t4
xor t1,t1,t5
xor t2,t2,t6
xor t3,t3,t7

bitrev.d t4,t0
bitrev.d t5,t1
bitrev.d t6,t2
bitrev.d t7,t3

bytepick.d t0,t6,t5,3
bytepick.d t1,t4,t7,3
bytepick.d t2,t5,t4,3
bytepick.d t3,t7,t6,3

bitrev.8b t4,t0
bitrev.8b t5,t1
bitrev.8b t6,t2
bitrev.8b t7,t3

ld.d t0,sp,64 
ld.d t1,sp,72
ld.d t2,sp,80 
ld.d t3,sp,88

xor t0,t0,t4
xor t1,t1,t5
xor t2,t2,t6
xor t3,t3,t7

addi.d a0,zero,1
addi.d a1,sp,32
li.d a2,64
li.d a7,64
syscall 0

li.d t4,64
clo.d t5,t0
bne t5,t4,fail
clo.d t5,t1
bne t5,t4,fail
clo.d t5,t2
bne t5,t4,fail
clo.d t5,t3
bne t5,t4,fail
b success

```



## 查阅指令手册分析 

```


龙芯指令集

output  030h 64bytes 

t0 - t7 这样一个线性表  每一个都是 64 bit  8bytes				
ld.d t0,sp,0 										1
ld.d t1,sp,8 
ld.d t2,sp,16 
ld.d t3,sp,24
ld.d t4,sp,32
ld.d t5,sp,40
ld.d t6,sp,48
ld.d t7,sp,56


xor 先异或运算
xor t0,t0,t4											2
xor t1,t1,t5
xor t2,t2,t6
xor t3,t3,t7

 
t0 = t0^t4
t1 = t1^t5
t2 = t2^t6
t3 = t3^t7

bitrev.d t4,t0  位反转 d 表示64位									3
然后 做逆序运算
t4 = t0[63:0]
t5 = t1[63:0]
t6 = t2[63:0]
t7 = t3[63:0]

bytepick.d 运算
bytepick.d t0,t6,t5,3    t5:t6 形成128bit的数据 从左侧数第3个字节(0开始)开始截取 8个字节(64bit) -->t0
bytepick.d t1,t4,t7,3 
bytepick.d t2,t5,t4,3    								4
bytepick.d t3,t7,t6,3 

bitrev.8b 操作
bitrev.8b t4,t0   t0的[7:0]逆序送入t4的[7:0]这样 每8 位一组 进行8次 刚好 64bit操作 (8bit反转)
bitrev.8b t5,t1
bitrev.8b t6,t2											5
bitrev.8b t7,t3

ld.d 运算 				---------
ld.d t0,sp,64 		 从SP+64的位置 移动 双字 32bit到 t0 ---->我感觉应该是 64 bit
ld.d t1,sp,72
ld.d t2,sp,80 
ld.d t3,sp,88
							这两个步骤就没用
xor运算
xor t0,t0,t4  				t0 = t0^t4  t0 = ~t4 
xor t1,t1,t5
xor t2,t2,t6
xor t3,t3,t7		--------------------


addi.d a0,zero,1      
addi.d a1,sp,32			 sp+32地址送入a1	
li.d a2,64
li.d a7,64		4			
syscall 0

li.d t4,64
clo.d t5,t0         		对t0中从第63位开始向第0位方向计量连续比特'1'的个数，送入t5
bne t5,t4,fail					判断t5 与 t4 是否相等 ，不等 faile
clo.d t5,t1
bne t5,t4,fail
clo.d t5,t2
bne t5,t4,fail
clo.d t5,t3
bne t5,t4,fail
b success




t4= [0x82,0x05,0xF3,0xd1,0x05,0xb3,0x05,0x9d]
t5= [0xa8,0x9a,0xce,0xb3,0x09,0x33,0x49,0xf3]
t6= [0xd5,0x3d,0xb5,0xad,0xbc,0xab,0xb9,0xb4]
t7= [0x39,0xce,0xa0,0xbf,0xd9,0xd2,0xc2,0xd4]

```



## 解析代码

```c
#include <stdio.h>

unsigned long long revd(unsigned long long num)
{
    char org[64] = {};
    for(int i = 0; i < 64; ++i)
    {
        unsigned long long mask = 1;
        mask <<= i;
        org[i] = (num & mask) >> i;
    }
    unsigned long long result = 0;
    for(int i = 0; i < 64; ++i)
    {
        result |= ((unsigned long long)(org[i]) << (63 - i));
    }
    return result;
}

unsigned long long rev8B(unsigned long long num)
{
    unsigned long long result = 0;
    for(int i = 0; i < 8; ++i)
    {
        unsigned long long tmp = (num & ((unsigned long long)0xff << (8 * i))) >> (8 * i);
        char org[8] = {};
        for(int j = 0; j < 8; ++j)
        {
            org[j] = (tmp & (1 << j)) >> j;
        }
        tmp = 0;
        for(int j = 0; j < 8; ++j)
        {
            tmp |= ((unsigned long long)(org[j]) << (7 - j));
        }
        result |= (tmp << (8 * i));
    }
    return result;
}

unsigned long long makemask(int count)
{
    unsigned long long result = 0;
    for(int i = 0; i < count; ++i)
    {
        result |= 0xff << (i * 8);
    }
    return result;
}

unsigned long long repick(unsigned long long left, unsigned long long right,int count)
{
    return ((left & makemask(count)) << 8 * (8 - count)) | (right >> 8 * count);
    //bytepick.d t0,t6,t5,3
    //bytepick.d rd,rj,rk,sa3
    //rd = {rk[8*(8-sa3):0],rj[63:8*(8-sa3)]}
    //rd = {rk[40:0],rj[63:40]}
    //t0 = {t5[40:0],t6[63:40]}
    //t1 = {t7[40:0],t4[63:40]}
    //t2 = {t4[40:0],t5[63:40]}
    //t3 = {t6[40:0],t7[63:40]}
}

int main()
{
    unsigned long long
    sp32=0x8205f3d105b3059d,
    sp40=0xa89aceb3093349f3,
    sp48=0xd53db5adbcabb984,
    sp56=0x39cea0bfd9d2c2d4,
    sp64=0xc513455508290500,
    sp72=0x006d621abb30b918,
    sp80=0xbc555b9f4c6f86a1,
    sp88=0x050d78ad181a626d;
    
    unsigned long long 
    t0 = 0,
    t1 = 0,
    t2 = 0,
    t3 = 0,
    t4 = 0,
    t5 = 0,
    t6 = 0,
    t7 = 0;

    t0 = t1 = t2 = t3 = ~0;
    t4 = sp64 ^ t0;
    t5 = sp72 ^ t1;
    t6 = sp80 ^ t2;
    t7 = sp88 ^ t3;
    
    t0 = rev8B(t4);
    t1 = rev8B(t5);
    t2 = rev8B(t6);
    t3 = rev8B(t7);

    t4 = repick(t1,t2,3);
    t5 = repick(t2,t0,3);
    t6 = repick(t0,t3,3);
    t7 = repick(t3,t1,3);
    
    t0 = revd(t4);
    t1 = revd(t5);
    t2 = revd(t6);
    t3 = revd(t7);
    
    t4 = sp32;
    t5 = sp40;
    t6 = sp48;
    t7 = sp56;
    
    t0 ^= t4;
    t1 ^= t5;
    t2 ^= t6;
    t3 ^= t7;
    
    printf("%.8s%.8s%.8s%.8s",(char*)&t0,(char*)&t1,(char*)&t2,(char*)&t3);
	return 0;
}

```

