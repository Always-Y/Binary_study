## [ACTF新生赛2020]usualCrypt

---

- Base64算法编码表变种
- [Base64编码原理链接](https://blog.csdn.net/wo541075754/article/details/81734770)

---

### 运行程序观察程序特征

运行程序，可见：

![图1](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/1.png)

由上图可见，程序简单明了，给一个提示字符串，让输入flag



### 查壳

![图2](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/2.png)

用查壳软件查壳可知，无壳，同时了解到使用VC编写，32位程序



### IDA 静态分析

通过终端界面提示的字符串，然后根据交叉引用，定位到关键函数，F5反汇编，得到如下图所示：

![图3](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/3.PNG)

该函数，主体逻辑较为简单，很容易分析出来，整个流程已在上图中通过注释表面，主要关键是查看分析 Sub_401080函数

进入该函数，可见函数如下图

![图4](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/4.PNG)

![图五](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/5.PNG)

![图6](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/6.png)

注意在上图中，逻辑较为复杂，大量，反复使用了unk_40E0A0数组，点击查看，该数组即为Base64的编码表 

**故推测该函数位Base64编码**

同时注意到该函数在第21行，有一个函数，进入查看

![图7](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/7.PNG)

由上面已知，unk_40E0A0数组即为base64的编码表，而该函数，对改变表，进行处理，由于参数已知，逻辑简单

按照该函数逻辑，易得修改后的编码表**ABCDEFQRSTUVWXYPGHIJKLMNOZabcdefghijklmnopqrstuvwxyz0123456789+/**



最后需要注意，该函数主题是变形得base64加密，整个过程结束返回时，又调用了一个函数，点击查看，逻辑简单容易分析得到时一个字符大小写转换得过程

![图8](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/8.PNG)



到此整个流程，已经清晰：

**将输入的字符，使用变形的Base64编码，然后将得到的编码字符串中大写字母转小写，小写字母转大写，再与给定的字符串对比，若相同，则输入为flag，否则不是**

**整个过程的逆向并不很困难，稍微麻烦的是变形的Base64的处理，在这个过程中base64编码表的原理，变形后的编码表，每个位置上的字符可能发生改变了，但是他的序号位置是不变的，找到序号位置，再反过来根据位置查找原始的base64编码表，拼凑起来可以得到原始的base64编码字符串，这个时候就容易解密了**



### 脚本

```c++
#include <iostream>
#include <string>
using  namespace std;

int main(){
	string str1="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
	string str2="ABCDEFQRSTUVWXYPGHIJKLMNOZabcdefghijklmnopqrstuvwxyz0123456789+/";
	char c;
	string  str ="zMXHz3TIgnxLxJhFAdtZn2fFk3lYCrtPC2l9";
	string s="";
	for(int i=0;i<str.size();i++){
		if(str[i]<='z'&&str[i]>='a'){ //小写变大写 
			str[i]-=32;
		}
		else if(str[i]<='Z'&&str[i]>='A'){	//大写变小写 
			str[i]+=32;
		}
		else{
			//其他不变 
		}
	}
	//cout<<str<<endl;
	for(int i=0;i<str.size();i++){ //处理后的字符串对应逆表变化 
		for(int j=0;j<64;j++){
			if(str[i]==str2[j])
				s+=str1[j];
		}
	} 
	cout<<s;
	return 0;
}

//s = ZmxhZ3tiQXNlNjRfaDJzX2FfU3VycHJpc2V9

```

### Flag

将s的值使用Base64在线解密，可得flag

**flag{bAse64_h2s_a_Surprise}**