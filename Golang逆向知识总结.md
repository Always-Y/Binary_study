# Golang 逆向知识总结

## 前言

现在越来越多的逆向题是`Go`语言类型，且很多恶意软件使用`Go`语言来编写，所以写了这样一个文档，来对这方面的知识做一个归纳总结。



## 符号表

对于逆向，首先一个比较大的问题就是符号表，有无符号表对程序逆向分析的影响很大；对于一般的`C/C++`等程序由于涉及的主要逻辑函数较少，且该语言为大家所熟悉，所以脱去符号表；影响不大；但是对于`Go`语言，由于每一个程序在IDA中得到的函数数量极大，无符号很难分析程序；

因此一般对于脱去符号表的程序，我们常使用`IDAGolangHelper`来进行符号表的恢复，或者直接使用`IDA pro 7.6`以上版本，做了对`Go`语言的支持



##  .GOPCLNTAB

[.gopclnta节段原理](https://docs.google.com/document/d/1lyPIbmsYbXnpNj57a261hgOYVpNRcgydurVQIyZOz_o/pub)-------------->由`golang`之父说明了该节段，主要是为了用于程序发生崩溃时，定位`crash`的位置与函数名

该段的结构如下:

```c
type _func struct {	
	entry   uintptr // start pc				--->偏移
	nameoff int32   // function name		--->函数名 
	args  int32 // in/out args size	
	frame int32 // legacy frame size; use pcsp if possible	
	pcsp      int32	
	pcfile    int32	
	pcln      int32	
	npcdata   int32	
	nfuncdata int32}
```

可以看到该段含有相关的**函数名和偏移信息**，因此可以根据该信息恢复函数名





## IDAGolangHelper原理

`IDAGolangHelper`这类工具主要是通过`.GOPCLNTAB`节区段来提取部分信息，用于恢复函数名

主要进行了三个步骤：

- 找到pclntab结构体
- 提取函数地址
- 查找函数名偏移量

```python
import idc
import idautils
import idaapi
import Utilsinfo = idaapi.get_inf_structure()
try:
	is_be = info.is_be()
except:    
    is_be = info.mf
    
lookup = "FF FF FF FB 00 00" if is_be else "FB FF FF FF 00 00"

def check_is_gopclntab(addr):    
    ptr = Utils.get_bitness(addr)    
    first_entry = ptr.ptr(addr+8+ptr.size)    
    first_entry_off = ptr.ptr(addr+8+ptr.size*2)    
    addr_func = addr+first_entry_off    
    func_loc = ptr.ptr(addr_func)    
    if func_loc == first_entry:        
        return True    
    return False

def findGoPcLn():    
    pos = idautils.Functions().next() # Get some valid address in .text segment    
    while True:        
        possible_loc = idc.FindBinary(pos, idc.SEARCH_DOWN, lookup) #header of gopclntab        
        if possible_loc == idc.BADADDR:            
            break        
        if check_is_gopclntab(possible_loc):            
            return possible_loc        
        pos = possible_loc+1    
   return None

def rename(beg, ptr, make_funcs = True):    
    base = beg    pos = beg + 8 #skip header    
    size = ptr.ptr(pos)    
    pos += ptr.size    
    end = pos + (size * ptr.size * 2)    
    print "%x" % end    
    while pos < end:        
        offset = ptr.ptr(pos + ptr.size)        
        ptr.maker(pos)         #in order to get xrefs        
        ptr.maker(pos+ptr.size)        
        pos += ptr.size * 2        
        ptr.maker(base+offset)        
        func_addr = ptr.ptr(base+offset)        
        if make_funcs == True:            
            idc.MakeUnknown(func_addr, 1, idc.DOUNK_SIMPLE)            
            idc.MakeFunction(func_addr)        
        name_offset = idc.Dword(base+offset+ptr.size)        
        name = idc.GetString(base + name_offset)        
        name = Utils.relaxName(name)        
        Utils.rename(func_addr, name)

```



## Fool tools

由上述分析`IDAGolangHelper`工具的原理可以指导，其对于函数名的恢复主要是通过`.gopclntab`,而该段则是用于辅助判断`Crash`的，所以一般用不到，如果我们对该段加以修改，即可保证正常运行的情况下，混淆函数。该实验在参考文献二中已经验证；

在上面的结果上我们进一步做假想，如果直接去掉`.gopclntab`回怎样？



## 参考文献

[利用Ghidra逆向分析Go二进制程序（上篇）](https://www.4hou.com/posts/8OJ2)

[利用Ghidra逆向分析Go二进制程序（下篇） - 嘶吼 RoarTalk – 回归最本质的信息安全,互联网安全新媒体,4hou.com](https://www.4hou.com/posts/0OxV)

[Golang function's name obfuscation : How to fool analysis tools?](https://tuanlinh.gitbook.io/ctf/golang-function-name-obfuscation-how-to-fool-analysis-tools)

[Reverse Engineering Go Binaries with Ghidra](https://cujo.com/reverse-engineering-go-binaries-with-ghidra/)

