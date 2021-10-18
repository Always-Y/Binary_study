### [2019红帽杯] childRE

---

### 知识点：

关键在于 UnDecorateSymbolName 函数的认识与学习

1. [UnDecorateSymbolName ](https://baike.baidu.com/item/UnDecorateSymbolName/12635968?fr=aladdin)   [函数](https://baike.baidu.com/item/函数/18686609)反修饰指定已修饰的 C++ 符号名。
2. [c/c++函数名修饰规则](https://blog.csdn.net/leehsiang1/article/details/86167500)



---

整个题 的运算流程，先是 后序遍历，然后使用上诉函数 对输入字符串后序遍历的得到的函数名解析，由31位变为62位







```C++
#include <iostream>
#include <string>
using namespace std;

int main(){
	string s1="(_@4620!08!6_0*0442!@186%%0@3=66!!974*3234=&0^3&1@=&0908!6_0*&";
	string s2="55565653255552225565565555243466334653663544426565555525555222";
	//长度都为62；
 	string ss ="1234567890-=!@#$%^&*()_+qwertyuiop[]QWERTYUIOP{}asdfghjkl;";
 	string outString="";
 	string str ="0123456789QWERTYUIOPASDFGHJKLZX";
 	string flag="";
 	char ch[31]={'Y','U', '7', 'I', 'O', '8', '3', 'P', 'A', 
	 '9', 'S', 'D', 'Q','4','1', 'F', 'G', 'W', 'H', 'J', 'E',
	  '5', 'K', 'L', 'R', 'Z', 'X', 'T','6', '2','0'};
	string v2="?My_Aut0_PWN@R0Pxx@@AAEPADPAE@Z"; //根据outString人工推到得到的
    //-------------------------------------------------------------
	 for(int i=0;i<62;i++){                  //第一部分 求outString
 		int v11=0,v12=0;
 		for(int j=0;j<23;j++){
 			if(s1[i]==ss[j])
 				v12=j;
 			if(s2[i]==ss[j])
 				v11=j;
 		}
 		outString+= char(v11*23+v12);
 	}
 	cout<<outString<<endl;
 	//private: char * __thiscall R0Pxx::My_Aut0_PWN(unsigned char *)
    //------------------------------------------------------------------------
	for(int i=0;i<31;i++){		//根据输入字符串后序遍历后的位置来推到v2各字符原来的位置得到Flag
		for(int j=0;j<31;j++){
			if(ch[j]==str[i])
				flag+=v2[j];
		}
	}
	cout<<flag; //Z0@tRAEyuP@xAAA?M_A0_WNPx@@EPDP
	return 0;
}
```

