---
layout: article
title:  ELF节和节头
tags: [C++,ELF,Section]
author: fatOrange
aside:
  toc: true
sidebar:
  nav: Binary_Analysis
---


### 节头

在介绍节头(section header)之前，我们在此指出段(segment)和(section)的区别，千万不能将这两个概念混为一谈。节，不是段。段是程序执行的必要组成部分，在每个段中，会有代码或数据被划分为不同的节。而节头表是对这些节的位置和大小的描述，主要用于调试和链接。节头对于程序的执行来说不是必需的，没有节头表，程序仍然可以执行。节头是对程序头的补充。`readelf -l` 命令可以显示一个段对应哪些节，可以很直观地看到节和段之间的关系。

如果二进制文件中缺少节头，并不意味着节就不存在。只是没有办法通过节头来引用节，对于调试器或者反编译程序来说，只是可以参考的信息变少了而已。

节头便于我们更细粒度地检查一个ELF目标文件的某部分或者某节。事实上，有了节头，一些需要使用节头的工具，如 objdump等，就能为逆向工程带来很多便利。如果去掉了节头表，就无法获取像.dynsym这样的节，而在.dynsym节中包含了描述函数名和偏移量/地址的导入/导出符号。

ELF头部结构：

```c++
typedef struct
{
  Elf64_Word    sh_name;/* offset into shdr string table for shdr name*/
  Elf64_Word    sh_type;/* Section type I.E SHT_PROGBITS*/
  Elf64_Xword   sh_flags;/* Section flags I.E SHT_WRITE|SHT_ALLOC*/
  Elf64_Addr    sh_addr;/* Section virtual addr where section begins */
  Elf64_Off     sh_offset;/* offset of shdr Section file */
  Elf64_Xword   sh_size;/* Section size in bytes */
  Elf64_Word    sh_link;/* Link to another section */
  Elf64_Word    sh_info;/* Additional section information */
  Elf64_Xword   sh_addralign;/* Section alignment */
  Elf64_Xword   sh_entsize;/* Entry size if section holds table */
} Elf64_Shdr;
```

### 常见节

#### .text节

保存了程序代码指令的代码节。一段可执行程序，如果存在PHDR .text节就会存在于text段中。由于.text节保存了程序代码,包括局部变量，因此的类型为SHT_PROGBITS.

#### .data节

保存了那些 **初始化了的全局静态变量和局部静态变量**。

#### .rodata节

```c++
printf("Hello World!\n");
```

这行代码中用到一个字符串常量 "Hello World!\n" 它是一种只读数据，所以它被放倒了.rodata中。.rodata中也一般是存放 **只读变量** (如const修饰的变量)和**字符串常量**，而且操作系统在加载的时候将 ".rodata" 属性映射为只读，对于这个段的任何操作都会作为非法处理，保证了程序的安全性。但是有的编译器也会将字符串常量保存在.data节中。

#### .bss节

.bss段存放的是 **未初始化的全局变量和局部静态变量** 。

```c++
static int x1 = 0 ;
static int x2 = 1 ;
```

其中x1会被放在.bss段中，x2会被放在.data段中。x1为0,可以认为是未初始化的，因为未初始化的全局变量都是0;所以被优化掉了，放在.bss。

### 段和节的布局

text段的布局:

- [.text]： 程序代码，临时变量。
- [.rodata]：只读数据
- [.hash]：符号散列表
- [.dynsym]：共享目标文件符号数据
- [.dynstr]：共享目标文件符号名称
- [.plt]：过程链接表
- [.rel.got]：G.O.T重定位数据

data段的布局:

- [.data]：全局和静态的初始化变量
- [.dynamic]：动态链接结构和对象
- [.got.plt]：全局偏移表
- [.bss]：全局和静态的未初始化变量

### 后话

还有其他一些节也很重要，你可以通过搜索关键字 "ELF 节头" 在搜索引擎上获取更多信息。