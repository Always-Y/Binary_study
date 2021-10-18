## T0p_Gear

#### 题目来源：

DASCTF 六月赛 第一题

---

#### 运行程序观察程序特征

**首先，要注意，该程序的题目是给了一个压缩包，解压后，有一个TXT文件和一个elf文件**

1.首先查看 txt文件

![图1](D:\Binary_Study\T0p_Gear\1.PNG)

一堆乱码，没啥有用信息，啥也得不到；然后放到winhex等任意一个二进制文件查看器看一下；

![图2](D:\Binary_Study\T0p_Gear\2.PNG)

emmm看完我表示，一脸懵，后来根据需求返回来才注意到这是128位(先提前注明)

因为txt文本基本看不出来啥，所以直接就 elf文件吧....................



2.运行程序提示，输入，随便输入几个，显示失败；如下图：

![图5](D:\Binary_Study\T0p_Gear\3.PNG)

很明显，程序有对输入的字符串 进行比对啥的处理

到这里，对程序的基本信息了解完毕；



#### 使用查壳工具进行查壳

使用查壳工具查壳，如下图所示：

![图4](D:\Binary_Study\T0p_Gear\4.PNG)

很明显，可以看的出来，是一个UPX壳，即压缩壳，需要脱壳，手工脱壳水平不够，放弃；

使用工具，在Ubuntu上下载 upx 使用命令： upx -d 文件名   即可脱壳；



#### IDA静态分析

将脱壳的程序拖入IDA64

![图5](D:\Binary_Study\T0p_Gear\5.PNG)

很明显，可以看到在IDA中，关键的字符串信息，已经显露出来，整个程序流程都很清楚 

F5反编译 

![图6](D:\Binary_Study\T0p_Gear\6.PNG)

上图中，程序整体基本就出来，输入之后进行三次对比，chk1，chk2，chk3；个人推测，这应该C++写的一个类的三个方法；当然对C++的反编译不是很熟悉，也不能肯定；

chk1进去后，得到下图

![图7](D:\Binary_Study\T0p_Gear\7.PNG)

很明显，给了一个字符串，对 sub_401080函数进行查看，发现就是一个字符串匹配，所以chk1的检查基本就出来，对字符串做一个对比；在这里要说明一下，通过查看HEX-view窗口，发现V3这里展示的字符串是内存中存储的字符串的逆向，形成的原因暂时还不确定，个人感觉应该是大小端序的问题，所以正确的字符串应该是**c92bb6a5**

运行程序，将该字符串输入，提示还要继续输入，此时，就应该对应chk2 函数，如下图

![图8](D:\Binary_Study\T0p_Gear\8.PNG)

从函数名称可以得到一些信息 AES ，File 似乎和开始的txt文件有了关系，进一步查看该函数

![图9](D:\Binary_Study\T0p_Gear\9.PNG)

分析该函数可知，他将 .txt 文件内容，读入，然后AES解密，然后去掉字符串中的字符‘f’，密钥即为上面的v3，v4，要注意v3，v4是逆序的 所以密钥为**4a5b6c7ffafc7011**

AES有多种加密模式，什么 这个只用到了密钥，所以是ECB模式，然后招python脚本解密 可得到字符串a6c30091ffffffff，去掉f得到**a6c30091**

chk2 基本到此结束



然后是chk3，从IDA中查看

![图10](D:\Binary_Study\T0p_Gear\10.PNG)

照抄改代码，分析得到

```c++
#include <iostream>
#include <string>
using namespace std;

int main(){
	string str=":<=>>l00:l<jk?mm";
	for(int i=0;i<16;i++){
		if(str[i]=='F'){
			str[i]=(i+3)^str[i]-10;
		}
		else
			str[i]^=8;
	}
	cout<<str;
	return 0;
}

```

解析的 字符串 24566d882d4bc7ee 输入

![图11](D:\Binary_Study\T0p_Gear\11.PNG)

