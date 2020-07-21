---
layout: article
title:  ELF程序头和段
tags: [C++,ELF,Segment]
author: fatOrange
aside:
  toc: true
sidebar:
  nav: Binary_Analysis
---


### 段和程序头

Unix 最早的文件可执行文件格式并不是 **COFF格式** 而是 a.out 格式，它的设计非常地简单，以至于后来共享库这个概念出现的时候，a.out 格式就不够用了，于是人们才提出了COFF格式，COFF的主要贡献是在文件目标文件里面引入了**“段”( segment )**机制，段在内核装载时被解析，描述了磁盘上可执行文件的内存布局以及如何讲内存布局映射到文件中。可以通过ELF表头中的**e_phoff(程序头偏移量)** 来得到程序表头。程序表头描述了可执行文件的(包括共享库)中的段及其类型。

ELF头部结构：

```c++
/* ELF Header */
typedef struct elfhdr {
	unsigned char	e_ident[EI_NIDENT]; /* ELF Identification */
	Elf32_Half	e_type;		/* object file type */
	Elf32_Half	e_machine;	/* machine */
	Elf32_Word	e_version;	/* object file version */
	Elf32_Addr	e_entry;	/* virtual entry point */
	Elf32_Off	e_phoff;	/* program header table offset */ 	 <<---
	Elf32_Off	e_shoff;	/* section header table offset */
	Elf32_Word	e_flags;	/* processor-specific flags */
	Elf32_Half	e_ehsize;	/* ELF header size */
	Elf32_Half	e_phentsize;	/* program header entry size */
	Elf32_Half	e_phnum;	/* number of program header entries */
	Elf32_Half	e_shentsize;	/* section header entry size */
	Elf32_Half	e_shnum;	/* number of section header entries */
	Elf32_Half	e_shstrndx;	/* section header table's "section 
					   header string table" entry offset */
} Elf32_Ehdr;
```

程序头类型有5种比较常见，在介绍这些之前，我们以ELF32_Phdr结构体为例(其中ph: program header 缩写)，看一下程序头的内部结构： 

```c++
typedef struct {
    Elf32_Word	p_type; /*segment type*/  <<---
    Elf32_Off	p_offset;/*segment offset*/
    Elf32_Addr	p_vaddr;/*segment virtual address*/
    Elf32_Addr	p_paddr;/*segment physical address*/
    Elf32_Word	p_filesz;/*size of segment in the file*/
    Elf32_Word	p_memsz;/*size of segment in memory*/
    Elf32_Word	p_flags;/*segment flags , I.E. execute|read|write*/
    Elf32_Word	p_align;/*segment alignment in memory*/
} Elf32_Phdr;
```

Elf32_Ehdr中 **e_phoff** **指定了 Elf32_Phdr**的所在位置(或者说"偏移")。

### 程序头类型

程序头主要描述了程序执行时在内存中的布局,这里的 PT 是 **p_type** 的缩写。在 Elf32_Phdr 结构中 p_type 有5种常见的类型。

#### PT_LOAD

一个可执行文件至少有一个 PT_LOAD 类型的段。这类程序头描述的是可装载的段，就是说，这种类型的段将被装载或映射到内存中。

例如，一个需要动态链接的 ELF 可执行文件通常包含以下两个可装载的 PT_LOAD 段。

存放程序代码的 **text 段**和存放全局变量的 **data 段**。这两个段会被映射到内存中，并根据 p_align 中存放的值在内存中对对齐。

通常将 text 段的权限 (p_flag) 设置为PF_X&#124PF_R (读和可执行)。

通常将 data 段的权限 (p_flag) 设置为 PF_W&#124PF_R (读和写)。注：PF 是 p_flag 的缩写

#### PT_DYNAMIC

动态段是动态链接可执行文件所特有的，包含了动态链接器所必需的一些信息。在动态段中包含了动态链接器所必需的一些内容信息。这个段主要包括.dynamic节。特殊符号 `_DYNAMIC` 用于标记包含以下结构的数组的节。

```c++
typedef struct
{
  Elf32_Sword	d_tag;			/* Dynamic entry type */
  union
    {
      Elf32_Word d_val;			/* Integer value */
      Elf32_Addr d_ptr;			/* Address value */
    } d_un;
} Elf32_Dyn;
```

- `d_val`

  这些目标文件表示具有各种解释的整数值。

- `d_ptr`

  这些目标文件表示程序虚拟地址。在执行过程中，文件虚拟地址可能与内存虚拟地址不匹配。对动态结构中包含的地址进行解释时，运行时链接程序会根据原始文件值和内存基本地址来计算实际地址。为确保一致性，文件不应包含用于**更正**动态结构中的地址的重定位项。

通常，每个动态标记的值决定了 `d_un` 联合的解释。借助此约定，第三方工具可进行更简单的动态标记解释。值为偶数的标记表示使用 `d_ptr` 的动态节项。值为奇数的标记表示使用 `d_val` 的动态节项，或该标记既不使用 `d_ptr`，也不使用 `d_val`。

| **Name**             | **含义**                                                     | **Value**    | `d_un`      | **Executable** | **Shared Object** |
| -------------------- | ------------------------------------------------------------ | ------------ | ----------- | -------------- | ----------------- |
| `DT_NULL`            |                                                              | `0`          | ignored     | mandatory      | mandatory         |
| `DT_NEEDED`          | 运行时需要链接的共享库                                       | `1`          | `d_val`     | optional       | optional          |
| `DT_PLTRELSZ`        | PLT重定位的信息                                              | `2`          | `d_val`     | optional       | optional          |
| `DT_PLTGOT`          | GOT的地址                                                    | `3`          | `d_ptr`     | optional       | optional          |
| `DT_HASH`            | 符号散列表的地址                                             | `4`          | `d_ptr`     | mandatory      | mandatory         |
| `DT_STRTAB`          | 字符串表的地址                                               | `5`          | `d_ptr`     | mandatory      | mandatory         |
| `DT_SYMTAB`          | 符号表的地址                                                 | `6`          | `d_ptr`     | mandatory      | mandatory         |
| `DT_RELA`            | 相对地址重定位表地址                                         | `7`          | `d_ptr`     | mandatory      | optional          |
| `DT_RELASZ`          | RELA表的字节大小                                             | `8`          | `d_val`     | mandatory      | optional          |
| `DT_RELAENT`         | RELA表条目的字节大小                                         | `9`          | `d_val`     | mandatory      | optional          |
| `DT_STRSZ`           | 字符串表的字节大小                                           | `10`         | `d_val`     | mandatory      | mandatory         |
| `DT_SYMENT`          | 符号表条目的字节大小                                         | `11`         | `d_val`     | mandatory      | mandatory         |
| `DT_INIT`            | 初始化函数的地址                                             | `12`         | `d_ptr`     | optional       | optional          |
| `DT_FINI`            | 终止函数的地址                                               | `13`         | `d_ptr`     | optional       | optional          |
| `DT_SONAME`          | 共享目标文件名的字符串偏移量                                 | `14`         | `d_val`     | ignored        | optional          |
| `DT_RPATH`           | 库搜索路径的字符串偏移量                                     | `15`         | `d_val`     | optional       | ignored           |
| `DT_SYMBOLIC`        | 修改链接器，在可执行文件之前的共享目标文件中搜索符号         | `16`         | ignored     | ignored        | optional          |
| `DT_REL`             | REL relocs表地址                                             | `17`         | `d_ptr`     | mandatory      | optional          |
| `DT_RELSZ`           | REL表的字节大小                                              | `18`         | `d_val`     | mandatory      | optional          |
| `DT_RELENT`          | REL表条目的字节大小                                          | `19`         | `d_val`     | mandatory      | optional          |
| `DT_PLTREL`          | PLT引用的RELOC类型 （RELA或REL）                             | `20`         | `d_val`     | optional       | optional          |
| `DT_DEBUG`           | 还未定义，为调试保留                                         | `21`         | `d_ptr`     | optional       | ignored           |
| `DT_TEXTREL`         | 缺少此项表明重定位只能用于可写段(例如data段)                 | `22`         | ignored     | optional       | optional          |
| `DT_JMPREL`          | 仅用于PLT的重定位条目地址                                    | `23`         | `d_ptr`     | optional       | optional          |
| `DT_BIND_NOW`        | 指示动态链接器在将控制权交给可执行文件之前处理所有的重定位   | `24`         | ignored     | optional       | optional          |
| `DT_INIT_ARRAY`      | 初始化函数的指针数组的地址                                   | `25`         | `d_ptr`     | optional       | optional          |
| `DT_FINI_ARRAY`      | 终止函数的指针数组的地址。                                   | `26`         | `d_ptr`     | optional       | optional          |
| `DT_INIT_ARRAYSZ`    | `DT_INIT_ARRAY` 数组的总大小（以字节为单位）                 | `27`         | `d_val`     | optional       | optional          |
| `DT_FINI_ARRAYSZ`    | `DT_FINI_ARRAY` 数组的总大小                                 | `28`         | `d_val`     | optional       | optional          |
| `DT_RUNPATH`         | 库搜索路径的字符串表偏移量                                   | `29`         | `d_val`     | optional       | optional          |
| `DT_FLAGS`           | 特定于此目标文件的标志值。                                   | `30`         | `d_val`     | optional       | optional          |
| `DT_ENCODING`        | 大于或等于 `DT_ENCODING`、小于或等于 `DT_LOOS` 的动态标记值遵循 `d_un` 联合的解释规则。 | `32`         | unspecified | unspecified    | unspecified       |
| `DT_PREINIT_ARRAY`   | 预初始化函数的指针数组的地址。此元素要求同时存在 `DT_PREINIT_ARRAYSZ` 元素。仅在可执行文件中处理该数组。如果该数组包含在共享目标文件中，则会被忽略。 | `32`         | `d_ptr`     | optional       | ignored           |
| `DT_PREINIT_ARRAYSZ` | `DT_PREINIT_ARRAY` 数组的总大小                              | `33`         | `d_val`     | optional       | ignored           |
| `DT_NUM`             | 用作数字                                                     | `34`         | `d_ptr`     | optional       | optional          |
| `DT_LOOS`            | 保留用于特定于操作系统的语义起始位置                         | `0x6000000D` | unspecified | unspecified    | unspecified       |
| `DT_HIOS`            | 保留用于特定于操作系统的语义结束位置                         | `0x6ffff000` | unspecified | unspecified    | unspecified       |
| `DT_LOPROC`          | 保留用于特定于处理器的语义起始位置                           | `0x70000000` | unspecified | unspecified    | unspecified       |
| `DT_HIPROC`          | 保留用于特定于处理器的语义结束位置                           | `0x7fffffff` | unspecified | unspecified    | unspecified       |

其中，对于初学者最重要的是前三项 `DT_NEEDED` 、`DT_PLTRELSZ` 、`DT_PLTGOT`  ，其余知道有这三项即可，在此先按下不表。

#### PT_NOTE

PT_NOTE 类型的段，保存了与特定供应商或者系统相关的附加信息，以便于其他程序对一致性、兼容性等进行检查。PT_NOTE指向.NOTE节。

#### PT_INTERP

指定要作为解释程序调用的以空字符结尾的路径名的位置和大小。对于动态可执行文件，必须设置此类型。此类型可出现在共享目标文件中。例如: **/lib64/ld-linux-x86-64.so.2** 这样一段字符指示调用程序的位置。

#### PT_PHDR

指定要作为解释程序调用的以空字符结尾的路径名的位置和大小。对于动态可执行文件，必须设置此类型。此类型可出现在共享目标文件中。此类型不能在一个文件中多次出现。此类型（如果存在）必须位于任何可装入段的各项的前面。

#### 工具

你可以同 `objdump -x` 或者 `readelf -l` 来显示各段的相关信息。

![UohZ7D.md.png](https://s1.ax1x.com/2020/07/21/UohZ7D.md.png)

![UohJHS.png](https://s1.ax1x.com/2020/07/21/UohJHS.png)

#### 后话

除了这五种常见程序头/段之外还有几种其他的程序头，可以看 ORACLE的文档[[见](https://docs.oracle.com/cd/E24847_01/html/E22196/chapter6-14428.html#chapter6-83432)]表7-25段类型

### 参考资料

[linux第三次实践：ELF文件格式分析](https://www.cnblogs.com/cdcode/p/5551649.html)

[ORACLE 链接程序和库指南](https://docs.oracle.com/cd/E24847_01/html/E22196/chapter6-14428.html#chapter6-42444)

