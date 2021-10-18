# ELF学习

+++

参考文献：

[ELF文件](https://ctf-wiki.org/executable/elf/structure/basic-info/)

+++

###  ELF 文件简介

所谓的对象文件有三类：

- 可重定位的对象文件(Relocatable file)

  由汇编器汇编生成`.o`文件，可经过链接器链接处理后生成 可执行的对象文件 ，或者 可被共享的链接文件 ；或者使用`ar`工具将众多的`.o`归档为`.a`静态库文件

- 可执行的对象文件(Executable file)

  最常见的文件类型，Linux存在两种可执行文件类型，除了这里所说的Executable object file ，还有就是 可执行脚本(如 shell脚本)，shell脚本，本身只是文本，但执行该文本的程序为可执行对象文件

- 可被共享的对象文件(Shared object file)

  这些就是所谓的动态库文件，即`.so`文件，。类似于Windows系统的DLL文件



