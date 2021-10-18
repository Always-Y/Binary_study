## IDA远程调试---连接Linux

### 起因

今天刷BUUCTF第三题 Reverse2，由于该程序石ELF格式，只能运行在Linux，动态调试需要GDB，对此我并不熟悉

因此，跟大佬学习了，将软件放在Linux上，而在本地使用IDA进行调试



### 步骤

#### 环境准备

1. 将IDA包中 dbgsrv文件下 linux_server 和linux_server64 (分别对应32位和64位)以及  调试程序 复制到 Linux上

   ![图1](D:\Binary_Study\IDA远程调试---连接Linux\1.PNG)

2. 在Linux上给与 上诉文件按 运行权限，并查看相应的文件信息![图2](D:\Binary_Study\IDA远程调试---连接Linux\2.PNG)

3. 由上图可以知道该调试软件是64位，所以执行上诉两个文件中的对应的一个文件![图3](D:\Binary_Study\IDA远程调试---连接Linux\3.PNG)

4. 选择左上菜单栏的Debugger中的Process Option ，填写相应的参数,选择remote linux debugger![图4](D:\Binary_Study\IDA远程调试---连接Linux\4.PNG)

5. 点击本地IDA运行 ，就可以知道是否够连接上![图5](D:\Binary_Study\IDA远程调试---连接Linux\5.PNG)

   如上图所示 链接成功，调试程序已经开始运行；

   



