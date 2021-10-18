# 网鼎杯-青龙组-2020 Jocker

---

### 知识点

- SMC--Self Modify Code 自修改代码

- 调堆栈平衡

  **Windows 使用 VirtualProtect()函数修改内存页属性，而Linux下使用mprotect修改属性。这两个函数就是SMC的标志**

---



### 运行程序观察特征

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/jocker_1.png)

由上图可知，该程序要求用户输入flag，然后在程序内部检查flag是否正确，并给定一个对于输出。



### 查壳

使用工具查壳，可知该程序无壳



### IDA 静态分析

将程序拖入IDA ，得到流程图：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Jocker_2.png)

由上图可知，输入字符串会计算长度与0x18，也就是24相比，如果不等与24 就会输入Wrong；

所以得到一个信息 **Flag长度为24**



#### 调堆栈不平衡

按下F5键查看反汇编代码 ，得到下图，可知程序堆栈不平衡

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Jocker_3.png)

**按空格键，调整为汇编代码形式，选择菜单栏的Option->General,勾选如下图**

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Jocker_4.png) 



然后可得：

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Jocker_5.png)

于 0x00401833处 Alt+K快捷键，将堆栈值调为0，可得下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Jocker_6.png)



F5键 可得 0x0040184C堆栈不平衡，与上面相同的方法可调，在F5键 可得到如下反汇编代码

#### SMC

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Jocker_7.png)

整个流程如上图所示 ,如果你通过wrong和omg函数计算出flag为**flag{fak3_alw35_sp_me!!}**，那么恭喜你，答错了，这是是一个迷惑性的Flag

**对SMC的处理**

使用 python脚本解密 ,具体脚本见解密程序部分

解密后，对IDA汇编进行修正，然后F5即可得到encrypt()函数



![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Jocker_8.png)

编写程序可得前19位



同样整理后，F5键可得finally函数

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Jocker_9.png)

分析上述finally函数，可知该函数无用，所以推测是XOR运算，最后五个字符与一个特定密钥XOR运算，由于flag最后一个字符位“}”是可以确定的，所以通过v7 与‘}’得到密钥 再与v3-v7进行异或运算得到flag的后五位

### 解密程序

SMC解密脚本：

```Python
for i in range(187):
    num = Byte(0x00401500+i)^0x41
    PatchByte(0x00401500+i,num)
    
print "success"
```



Flag解密程序

```C++
#include <iostream>
#include <string>
using namespace std;

int main(){
	int v1[19]={0xe,0xd,0x9,0x6,0x13,0x5,0x58,0x56,0x3e,
	0x6,0xc,0x3c,0x1f,0x57,0x14,0x6b,0x57,0x59,0xd};
	string s ="hahahaha_do_you_find_me?";
	string flag="";
	int num[5] = {37,116,112,38,58};
	int key;
	for(int i=0;i<19;i++){
		char c = s[i]^v1[i];
		flag+=c;
	}
	cout<<flag<<endl;
	key = num[4]^'}';
	for(int i=0;i<5;i++){
		flag+= key^num[i];
	}
	cout<<flag;
	return 0;
}
```

