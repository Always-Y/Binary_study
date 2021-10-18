# N1CTF 2020 oflo

---

### 知识点

- 花指令

[题源](https://github.com/Nu1LCTF/n1ctf-2020/tree/main/RE/oflo)

---



### 运行程序查看程序特征

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/oflo_1.png)

由上图可知，一开始输出一串关于系统信息的字符串，然后等待用户输入，任意输入几个字符，得到“Segmentation fault”的反馈



使用file指令可查看文件信息 如下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/oflo_2.png)

由上图可知该程序为64位的ELF文件，已去掉符号表



### 查壳

使用工具可知，无壳



### IDA静态分析

将oflo拖入IDA64，经过分析，可得到下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/oflo_3.png)



如上图所示，IDA分析得到该程序的启动流程，并标注出main函数，上图红框标记。双击可进入main函数，得到如下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/oflo_4.png)



汇编代码整理：

首先将数据都转为代码段，对于为正常识别的数据选中后按  c 键即可标识为代码 可得到下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/oflo_5.png)

上图中 db 0E8h, 0EBh, 12h,后面多次出现，转换成代码后如上图，可以知道存在不合理地址所以，很明显存在花指令



同时发现0x0000000400CCF处有大量数据，c键转化为代码 





花指令Patch：

将上述标红的部分Nop 后 修正 F5  仍然报错 0x400BB7 堆栈对战不平衡

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/oflo_6.png)



同理，可观察改修掉0x400CB5处代码  整理后得到如下图

![](https://ms-study.oss-cn-chengdu.aliyuncs.com/Binary_study/RE/oflo_7.png)



### 对应解体脚本

对sub_400A69前10个字节进行动态patch，异或的内容是输入的前5个字符，根据flag的格式，猜测是n1ctf，IDAPython脚本：

```python
a='n1ctf'
for i in range(10):
    PatchByte(0x400a69+i,Byte(0x400a69+i)^ord(a[i%5]))
    print'success'
```

求出flag：

```python
a = [53, 45, 17, 26, 73, 125, 17, 20, 43, 59, 62, 61, 60, 95]
s = ''
for i in range(14):
    s += chr((ord('Linux version '[i]) + 2) ^ a[i])
print('n1ctf' + s)
```

