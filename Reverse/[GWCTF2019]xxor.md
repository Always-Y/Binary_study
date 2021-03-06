# [GWCTF 2019]xxor

---

**重点：**

- Z3求解器的使用
- 数据类型的把握

---

## 运行程序观察程序特征

运行程序，得到如下图所示：

![图1](D:\Binary_Study\[GWCTF 2019]xxor\1.PNG)

由上图可以了解到以下几点：

- 会有六次输入
- 输入为只能是数字，若为字符，会出错
- 关键提示字符串 为 **Let us play a game** ，**you have six chances to input** ，**come on！**



## 程序查壳

使用PEiD或者 DectItEasy查壳，可以发现**该程序无壳**，在这里就不展示了，该文件为elf文件格式，拖到ubuntu，可得到如下图所示：

![图2](D:\Binary_Study\[GWCTF 2019]xxor\2.png)

可进一步知道该程序为64位



## IDA 静态分析

1. shift+F12通过搜索上诉字符串，查看交叉引用，定位到关键位置，得到如下图所示：

![图3](D:\Binary_Study\[GWCTF 2019]xxor\3.png)



主要过程，如图中所示，基本流程是将输入的六个数，分奇偶项，奇数项作为参数传入sub_400686处理，然后再对处理后的结果，作为参数输入sub_40070函数，进行处理判断；

#### sub_40070()函数

由于要逆向该程序，由结果得到输出，所以，我们要逆向分析该函数的逻辑流程，先分析sub_40070()函数，进入该函数，得到如下图所示：

![图4](D:\Binary_Study\[GWCTF 2019]xxor\4.PNG)

分析该函数，就知道这是一个**整数数组**，长度为6，要满足以下条件：

- a[1]=3746099070,a[5]==0x84F30420,a[1]=0X20AACF4
- a[2]-a[3]=2225223423 , a[3]+a[4]=4201428739 , a[2]-a[4]=1121399208 这就是一个方程组，将其解出来，可以手算或者使用Z3求解器

综上的 得到 6个值 ：

0. 0xDF48EF7E
1. 0x20CAACF4
2. 0xe0f30fd5
3. 0x5c50d8d6
4. 0x9e1bde2d	
5. 0x84F30420

到此，sub_40070()函数流程基本分析清楚了

#### sub_400686()函数

接下来是sub_400686()函数，首先是注意传入的参数，其中一个在前面已经提到了，是输入数组的下标偶数项，第二个参数，可以点击查看，知道是固定数据；

进入该函数：

![图5](D:\Binary_Study\[GWCTF 2019]xxor\5.PNG)

由上图可知，传入的虽然只有，下表为偶数项的数据，但是根据数组的索引方法，也是对下表为奇数项进行了处理；

**在对这个for循环分析的时候，要抓住，他的迭代过程中与自省的变化无关，在对v3处理时，v4的值是固定的，只有v5的值是变化的，但v5的值的变化规律又是很好跟踪的，同样在对V4处理的过程中，v3是不变的**

根据该函数编写脚本时，有几点值得注意：

1. v5在迭代中，必然溢出，不用管他，但是数据的定义类型一定要注意和该函数中一直，否则在处理中会得到不一样的值
2. 该函数中v3，v4定义为unsigned int ，即说明输入的数组都是无符号整型，在定义相关变量是也需要一致(不能为long long之类的)，否则会出错；



## 脚本

```c++
#include<iostream>

using namespace std;

int main(){
	unsigned int num[6]={0xDF48EF7E,0x20CAACF4,0xe0f30fd5,
	0x5c50d8d6,0x9e1bde2d,0x84F30420};
	unsigned int result[6]={0};
	int  v5;
	for(int j=0;j<=4;j+=2){
		v5 = 1166789954*0x40;
		unsigned int v3 = num[j];
		unsigned int v4 = num[j+1];
		for(int i=0;i<64;i++){ //a2 2 2 3 4 
			v4-=(v3+v5+20)^((v3<<6)+3)^((v3>>9)+4)^0x10;
			v3-=(v4+v5+11)^((v4<<6)+2)^((v4>>9)+2)^0x20;
			v5-=1166789954;			
		}
		result[j+1]=v4;
		result[j]=v3;
	}
	for(int j=0;j<6;j++){
		//cout<<hex<<result[j]<<endl;	
		cout<<*((char*)&result[j]+2)<<*((char*)&result[j]+1)<<*((char*)&result[j]);	}
	return 0;
}
```

