# Pwn_Canary 学习

### 源程序

```C
#include<stdio.h>
int main(){
    char *name = "Canary Test";
    printf("My name is %s");
    return 0;
}
```



### 编译 

#### 首先是 没有canary 和PIE 保护来编译

> gcc -no-pie -fno-stack-protector -m32 -o Canary Canary.c

得到下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Canary_1.png)

上图中，有以下几点注意：

- and esp,0xfffffff0 是使得栈指针对齐，最后一位为0，按16字节，便于寻址访问，加快速度

#### 具有Canary保护

> gcc -no-pie -fstack-protector-all -m32 -o Canary Canary.c

-all 表示对所有的函数保护代码 ，不加 -all参数，即只为局部变量中含有 char 数组的函数插入保护代码

得到如下图：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Canary_2.png)

