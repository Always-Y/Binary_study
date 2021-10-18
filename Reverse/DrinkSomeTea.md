# # drinkSomeTea

---

该题为Mar-Dasctf三月杯u，看名字就知道和Tea算法有关

---



### 运行程序

运行程序查看程序流程

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-04-10_10-42-45.png)

由上图可以得知，该程序运行即退出；不太合理；按照正常程序，应该有输入比对；



### 查壳

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/Snipaste_2021-04-10_11-00-01.png)

由此，可知该程序为32位，程序无壳。



### 静态分析



先留坑，以后再补起来。





### 算法

```c
#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>
#include <string.h>
void decrypt(int32_t* v, int32_t* k);
int main(){
	FILE *fp =NULL;
	int len_v = 0xE500;
	int32_t k[4];
	int32_t v[60000] ;
	int len_block = len_v/sizeof(int32_t);
	//int k[4]={0x67616C66,0x6B61667B,0x6C665F65,0x7D216761};
	char str[] ="flag{fake_flag!}";
	fp = fopen("D:/Binary_Study/Mar_DASCTF/drinkSomeTea的附件/tea.png.out","rb");
	fread(v,sizeof(int32_t),len_block,fp);
	fclose(fp);
	memcpy(k,str,16);
	for(int i=0;i<len_block;i+=2){
		//decrypt(v+i, k);
		decrypt(v+i, (int *)str);
		printf("%d\n",i); 
	}
	fp= fopen("D:/Binary_Study/Mar_DASCTF/drinkSomeTea的附件/tea.png","wb");
	fwrite(v,sizeof(int32_t),len_block,fp);
	fclose(fp);
	return 0;
}



void decrypt(int32_t* v, int32_t* k) {
    int v0=v[0], v1=v[1], sum=0xC6EF3720, i;  /* set up */
    int32_t delta=0x9e3779b9;                     /* a key schedule constant */
    int32_t k0=k[0], k1=k[1], k2=k[2], k3=k[3];   /* cache key */
    for (i=0; i<32; i++) {                         /* basic cycle start */
        v1 -= ((v0<<4) + k2) ^ (v0 + sum) ^ ((v0>>5) + k3);
        v0 -= ((v1<<4) + k0) ^ (v1 + sum) ^ ((v1>>5) + k1);
        sum -= delta;                                   
    }                                              /* end cycle */
    v[0]=v0; v[1]=v1;
}
```

