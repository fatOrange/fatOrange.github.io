---
layout: article
title: efn2头文件相关分析
tags: [rna, secondary structure predict, 代码分析]
author: fatOrange
aside:
  toc: true
sidebar:
  nav: secondary_prediction
mathjax: true
mathjax_autoNumber: true
---

# efn2文件分析

#### 模块分层

| 层次   | 内容  |
| ------ | ----- |
| 第一层 |       |
| 第二层 |       |
| 第三层 | RNA.h |
| 第四层 | efn.h |

### RNA.h分析

RNA类为RNA结构的所有单个序列操作提供了切入点。
请注意，风格化注释为通过doxygen自动记录提供了便利。
可以将以下常量提供给RNA类构造函数RNAInputType参数，以指示filepathOrSequence参数表示的文件类型

| 数字 | 意义                                                         |
| ---- | ------------------------------------------------------------ |
| 0    | 字符串 (不是一个文件)                                        |
| 1    | 结构文件 (*CT，DBN/DOT/BRACKET*)                             |
| 2    | 序列文件 (*SEQ，PASTA，*或者*文本*)                          |
| 3    | *Partition-Function-Save-File*  *(PFS)*                      |
| 4    | *Folding-Save-File (FSV or SAV)*                             |
| 5    | *Bracket 文件 (DBN/DOT/BRACKET) (1 也行,但是效率没那么高. 5 更严格一点*） |

宏定义：

typedef int RNAInputType; 只是为了帮助新程序员找到正确的常量（定义如下）

```c++
//file :RNA_class > RNA.h
//! A string containing the sequence -- NOT a file name.
#define SEQUENCE_STRING 0
//! CT Structure file
#define	FILE_CT 1
//! Sequence file (FASTA, SEQ or plain-text sequence)
#define	FILE_SEQ 2
//! Partition function save file (*.pfs)
#define	FILE_PFS 3
//! Folding Save file (*.sav)
#define	FILE_SAV 4
//! Dot-Bracket Notation Structure file (*.dbn, *.dot, *.bracket)
#define	FILE_DBN 5
```

#### RNA类

| 作用域 | 函数名/变量名                                     | 描述       | 参数 | 返回值 |
| ------ | ------------------------------------------------- | ---------- | ---- | ------ |
| public | RNA(const char sequence[], const bool IsRNA=true) | 构造函数<> |      |        |
| …      |                                                   |            |      |        |
| …      |                                                   |            |      |        |
|        |                                                   |            |      |        |
|        |                                                   |            |      |        |
|        |                                                   |            |      |        |
|        |                                                   |            |      |        |



### basepair.h分析

#### basepair类

| 作用域  | 函数名/变量名                       | 描述                        | 参数 | 返回值 |
| ------- | ----------------------------------- | --------------------------- | ---- | ------ |
| public  | basepair()                          | 一个空的构造函数            |      |        |
| …       | basepair(const int i, const int j)  | 带参的构造函数 i必须是j的5‘ |      |        |
| …       | int Get5pNucleotide()               | 获取5'nuc的索引             |      |        |
| …       | int Get3pNucleotide()               | 获取3'nuc的索引             |      |        |
| …       | void Set5pNucleotide(const int nuc) | 设置5'nuc的索引             |      |        |
| …       | void Set3pNucleotide(const int nuc) | 设置3'nuc的所有             |      |        |
| private | int p5,p3;                          | *5' and 3' nucs*            |      |        |

#### 声明

| 函数名                                  | 作用                                  |
| --------------------------------------- | ------------------------------------- |
| bool operator==(basepair a, basepair b) | 建立两个碱基对的比较                  |
| bool operator<(basepair a, basepair b)  | 如果a的5'小于b的5’ 那么a<b            |
| bool operator==(int a, basepair b)      | 如果5'或3'nuc等于i，则i等于一个碱基对 |
| bool operator==(basepair b, int a)      | 如果5'或3'nuc等于i，则i等于一个碱基对 |

### efn2.h头文件分析

#### efn2Interface类

| 作用域  | 函数名/变量名                                                | 描述                                                         | 参数                                       | 返回值                                        |
| ------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------ | --------------------------------------------- |
| public  | efn2Interface                                                | 名称:构造函数 <br>描述:初始化所有的私有化变量                |                                            |                                               |
| …       | bool parse( int argc, char** argv )                          | 名称:parse <br>描述:解析命令行的参数，确定哪个计算需要哪个选项 | argc:命令行参数的数量 <br> argv:命令行参数 | 如果解析完成没有错误，返回真； <br>否则返回假 |
| …       | void run()                                                   | 名称:run  <br>描述:运行所有的计算                            |                                            |                                               |
| private | calcEnergy(RNA*strand, const int structurenumber, const bool simpleMBRules) |                                                              |                                            |                                               |
| …       | string calcType                                              | 计算类型的描述                                               |                                            |                                               |
| …       | string ctFile                                                | 输入文件名，cf 文件类型                                      |                                            |                                               |
| …       | string outFile                                               | 输出文件名，能量文本文件                                     |                                            |                                               |
| …       | double intercept                                             | SHAPE约束的截距                                              |                                            |                                               |
| …       | string alphabet                                              | 核酸的类型(rna,dna或自定义)                                  |                                            |                                               |
| …       | string SHAPEFile                                             | 可选的SHAPE约束文件                                          |                                            |                                               |
| …       | double slope                                                 | SHAPE约束的斜率                                              |                                            |                                               |
| …       | bool stdPrint                                                | 表示输出是否应通过管道传输到标准输出以及已写入（真）或否（假）的标志 |                                            |                                               |
| …       | double temperature                                           | 计算的温度                                                   |                                            |                                               |
| …       | bool writeTherm                                              | 标志位，指示是否应写入热力学详细信息文件（true）或不写入（false）。 |                                            |                                               |
| …       | bool simple                                                  | 标志着是否应该使用简单的能量规则，即符合动态编程算法的规则。 |                                            |                                               |
| …       | bool quiet                                                   | 抑制不必要的输出                                             |                                            |                                               |
| …       | double pseudoP1, pseudoP2                                    | 计算包含假结能量的参数                                       |                                            |                                               |
| …       | double pseudo_params[16]                                     | 伪结惩罚计算常数的数组。                                     |                                            |                                               |
| …       | bool omitErrors                                              |                                                              |                                            |                                               |
| …       | string countFile                                             | 输出参数使用次数的文件。                                     |                                            |                                               |

