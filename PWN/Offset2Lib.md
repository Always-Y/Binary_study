# Offset2Lib

---

本文即为，学习Pwn的blog阅读笔记，原文见如下链接

https://cybersecurity.upv.es/attacks/offset2lib/offset2lib.html#intro

https://cybersecurity.upv.es/solutions/aslrv2/aslrv2.html

[翻译](https://cloud.tencent.com/developer/article/1036500)

---



### Introduction

​		ASLR 是一种同于防止攻击者通过了解目标代码或者数据加载地址从而进行攻击的保护手段；因此ASLR的效果取决于攻击者不了解整个地址空间布局。所以PIE+ASLR才能发挥最大防护效果



### ASLR Weakness

​		该脆弱性仅对于 GNU/Linux 有用，是ASLR设计上的弱点，在64bit系统可以很轻易地修复







------------------------------------------------------------------------

==没说好说的，说实话，虽然看完了全文，每句话都知道啥意思，就是搞不懂他在干嘛，先留坑吧==