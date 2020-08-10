---
layout: article
title: ELF文件类型
tags: [C++,ELF]
author: fatOrange
aside:
  toc: true
sidebar:
  nav: Binary_Analysis
---

### 可执行文件格式类型

现在 PC 平台流行的**可执行文件格式(executable)**主要是 windows 下的 PE(Portable Executable) 和Linux 下的 ELF(Executable Linkable Format) ,他们都是 **COFF(Common file format)** 的变种。目标文件和可执行文件的结构相似，可以和可执行文件格式一起存储(见图)。广义来说，可以讲这两文件看作一种文件。在 windows 下称他们为 PE-COFF 文件格式，Linux 下称为 ELF文件。

![URIk5t.png](https://s1.ax1x.com/2020/07/19/URIk5t.png)

COFF 是由Unix System V Release 3 首先提出并且使用的格式，微软公司基于 COFF 制定了 PE 格式标准，并用于 WindowsNT 系统中，System V Release 4 给予 COFF 制定了ELF格式，并沿用至今(System V Release 5)。因为PE和ELF格式都是源自于同一种可执行文件格式 COFF，所以他们如此相似。

通过编译生成文件的用途多种多样，一个ELF文件可以被标记为一下几种类型之一：

| ELF文件类型 | 说明                                                         | 例子                                |
| ----------- | ------------------------------------------------------------ | ----------------------------------- |
| ET_NONE     | 未知类型。这个标记表明文件类型不确定，或者还未定义。         |                                     |
| ET_REL      | 重定位文件。ELF类型标记为 Relocatable File ,意味着该文件被标记为一段可重定位的代码，可以被用来链接成可执行文件或共享目标文件。 | Linux:xx.o<br> windows:xx.obj       |
| ET_EXE      | 可执行文件。ELF类型为Executable File。是一个进程开始执行的入口。 | Linux:helloworld<br> windows:xx.exe |
| ET_DYN      | 共享目标文件。ELF类型为 Shared Object File，该文件是一个可动态链接的文件，也成为共享库，这类共享库会在程序运行时被装载并链接到程序的镜像中。可以在以下两种情况使用：<br> 一是链接器可以使用共享目标文件和其他重定位文件产生新的目标文件。<br> 二是链接器可以使用多个共享目标文件和可执行文件结构，作为进程影像的一部分来执行。 | Linux: xx.so<br> windows:xx.DLL     |
| ET_CORE     | 核心文件。ELF类型为 Core Dump File 当程序意外终止，会在核心文件中记录整个进程的镜像信息。可以使用GDB读取这类文件来辅助调试并查找程序崩溃的原因。 | linux : core dump                   |

在 Linux 下，使用 `file` 命令或者`readelf -h` 来查看相应的文件格式，上面几种文件格式在这两个命令下会显示出相应的格式。

使用`readelf -h` 除了可以查看文件格式，还可以看到原始的 ELF 文件头。ELF 文件头从文件的0偏移量开始，标记了 ELF 文件类型、结构和程序开始执行的入口地址，并提供了其他 ELF 头(节头和程序头)的偏移量。

### 结构和输出

ELF头部结构：

```c++
/* ELF Header */
typedef struct elfhdr {
	unsigned char	e_ident[EI_NIDENT]; /* ELF Identification */
	Elf32_Half	e_type;		/* object file type */
	Elf32_Half	e_machine;	/* machine */
	Elf32_Word	e_version;	/* object file version */
	Elf32_Addr	e_entry;	/* virtual entry point */
	Elf32_Off	e_phoff;	/* program header table offset */
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

这里只是展示ELF的作用是了解 `readelf -h` 是如何输出的，具体的会在下面讲解。

![Uo7dCF.png](https://s1.ax1x.com/2020/07/21/Uo7dCF.png)

### 文件头

我们上面输出的结果就是文件头，ELF的文件头，它包括:**ELF魔数，ELF文件类型，字节序，ELF版本，运行平台，ABI版本，ELF重定位类型，硬件平台，硬件平台版本，入口地址，程序头入口地址，段表位置，段的长度和段的数量。** 

他的数据结构通常被存储在**"/usr/include/elf.h"**中。大部分平台通用，有32位和64位两个版本，通过结构体名可以看出来，上一节展示的就是32位版本。32位版本和64位版本区别在于地址大小不同，有兴趣的朋友可以自行比较一下。比较例外的是 `e_ident` 这个成员对应了readelf输出结果中的"Magic","Class","Data","Version","OS/ABI","ABI Version"这6项，总体上参数都和输出的都是一一对应的。

#### 魔数

细心的你可能会发现上面结构体中第三行   `unsigned char	e_ident[EI_NIDENT]; /* ELF Identification */`  是一串固定的字符数组，怎么用他来表示6个不同的成员变量呢？

如果我们将 `Magic` 魔数单独拿出来分析，可能会发现其中的奥秘:

```sh
7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
```

这一串"难以理解"的16进制数字都是有意义的，最开始4个字节是所有ELF文件都必须相同的标识码，`0x7F`  代表ASCII 字符里面的DEL控制符，`0x45`、`0x4c`、`0x46`刚好是'E','L','F'三个字符的 ASCII 码。接下来一位，即第5位，表示 **ELF文件类型**，0:32位；1:64位。第6位，**字节序**，字节序，0:无效格式；1:小端格式；2: 大端格式。第7位，**ELF版本**，一般是1.剩下的9位没有标准定义一般为0，有些平台会将其作为拓展位。

魔数一般都是用来确认文件类型的，操作系统会在文件加载的时候确认文件是否正确，如果不正确会拒绝加载。

### 参考文献

https://refspecs.linuxbase.org/elf/gabi4+/

linux二进制分析

程序员的自我修养——链接、装载与库

