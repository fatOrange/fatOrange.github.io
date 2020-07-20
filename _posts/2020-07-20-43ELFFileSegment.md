---
​---
layout: article
title: ELF程序头——段
tags: [C++,ELF,Segment]
author: fatOrange
aside:
  toc: true
sidebar:
  nav: Binary_Analysis
​---
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
	Elf32_Off	e_phoff;	/* program header table offset */ 						<<---
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
    Elf32_Word	p_type; /*segment type*/
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

在 Elf32_Phdr 结构中 p_type 有5种常见的类型，也叫程序头类型。



