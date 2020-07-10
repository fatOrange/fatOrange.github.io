---
layout: article
title: Thermodynamics类分析
tags: [rna, secondary structure predict, 代码分析,Thermodynamics]
author: fatOrange
aside:
  toc: true
sidebar:
  nav: secondary_prediction
---

Thermodynamics类 用于封装打开和保存RNAstructure热力学参数的函数

**\#define TOLERANCE 0.01**  

TOLERANCE 是从310.15 K开始的最大偏差，在此之前将从磁盘读取焓参数以从310.15 K调整自由能变化。

------

Thermodynamics类

# public

## 初始化方法

### Thermodynamics(const bool isRNA=true, const char* const alphabetName=NULL, const double temperature=310.15)

#### 摘要：

#### 详细：

#### 返回值：

### Thermodynamics(const Thermodynamics& copyThermo)

#### 摘要：

#### 详细：

#### 返回值：

## 与折叠温度有关的功能

### int SetTemperature(double temperature)

#### 摘要：

将折叠温度设置为K。

#### 详细：

该功能允许用户指定310.15 K（37摄氏度）以外的折叠温度。<br> 这改变了折叠自由能的变化，该自由能变化将返回给现有结构并且将改变预测的结构集。<br> 调用此函数后，立即从磁盘读取热力学参数文件。 这些参数包括焓参数（.dh文件）和310.15处的自由能变化（.dat文件）。 文件必须位于$ DATAPATH环境变量指示的位置或pwd中。 更改温度只会更改后续计算。 例如，如果调用了结构预测方法，则在调用SetTemperature时不会更改一组预测结构。 同样，如果需要非默认值310.15 K的温度，则在调用结构预测方法之前必须调用SetTemperature。<br> 该函数返回错误代码，其中0为无错误，并且继承类中的GetErrorMessage（）或GetErrorMessageString（）可以解析非零错误。<br>

#### 参数： 

*\param temperature* 是浮点数，表示以K为单位的折叠温度。

#### 返回值：

一个整数，指示用于读取热力学参数的错误代码。 0 =没有错误。 5 =找不到文件。

### double GetTemperature() const

#### 摘要：

获取当前的折叠温度（单位为K）。

#### 详细：

#### 返回值：

指示折叠温度（以K为单位）的双精度数。

### string GetAlphabetName() const

#### 摘要：

获取应为其加载热力学参数的扩展字母的名称。

#### 详细：

#### 返回值：

## 从磁盘读取参数的函数

### int ReadThermodynamic(const char *directory=NULL, const char *alphabet=NULL, const double temperature=-1.0)

#### 摘要：

读取热力学参数的功能。

#### 详细：

此功能取决于温度（当前温度），以确定折叠时需要将自由能设置为不同于文件中读取的310.15 K的自由能。<br> 返回零=>没有错误，返回非零表示错误。<br> 需要热力学参数的公共功能会自动调用此功能。<br> 默认情况下，从\$DATAPATH环境变量中获取热力学参数的路径。<br> 如果需要特定路径，则通过在此处显式指定路径名作为参数来覆盖\$DATAPATH。 

#### 参数：

*\param* *directory* 是指向cstring的指针，该cstring指示包含热力学参数文件的目录。 默认情况下，它为NULL，并查询环境变量$ DATAPATH以获取此路径。

*\param* *alphabet* 是指向cstring的指针，该cstring指示核酸字母的名称（例如“ rna”，“ dna”等）；

*\param* *temperature* 指定自定义温度。 如果此参数小于零，则将其忽略。 否则，将重新计算自由能参数以应用于指定温度。

#### 返回值：

*\return* 指示是否发生错误的int值。

### int ReloadDataTables(const double new_temperature=-1.0)

#### 摘要：

使用原始字母名称从原始目录重新加载自由能数据表。

#### 详细：

#### 参数：

*\param* *temperature* 指定自定义温度。 如果此参数小于零，则将其忽略。<br> 否则，将重新计算自由能参数以应用于指定温度。

#### 返回值：

### bool VerifyThermodynamic()

#### 摘要：

强制读取尚未读取的数据表。 如果已加载表或尝试（重新）打开它们成功，则返回true。

#### 详细：

#### 返回值：

## Functions that provide accessibility of the underlying tables:

### datatable *GetDatatable()

#### 摘要：

在继承过程中使用此函数提供对自由能量更改参数的访问。<br> 此函数不生成任何错误代码。 （在构建过程中对此进行了错误检查）。

#### 详细：

#### 返回值：

指向具有自由能变化参数的数据表的指针。

### datatable *GetEnthalpyTable(const char* alphabet=NULL)

#### 摘要：

此函数用于提供焓表。

#### 详细：

如果从磁盘读取表时出错，则此函数将返回NULL指针。<br>

#### 参数：

*\param* *alphabet* 指定要使用的字母。 如果为NULL（默认值），则使用当前加载的字母（如果有）。 否则，将根据isrna的值使用默认的RNA或DNA字母。

#### 返回值：

具有焓变参数的指向数据表的指针。

### void ClearEnergies()

#### 摘要：

清除当前加载的能源数据表并释放其资源。

#### 详细：

#### 参数：

#### 返回值：

### void ClearEnthalpies()

#### 摘要：

清除当前加载的焓数据表并释放其资源。

#### 详细：

#### 参数：

#### 返回值：

### bool GetEnergyRead() const

#### 摘要：

返回此热力学实例是否填充了参数（从磁盘还是从另一个热力学类）。

#### 详细：

#### 参数：

#### 返回值：

一个布尔值，指示是否填充参数（true =是）。

### bool IsAlphabetRead() const

#### 摘要：

#### 详细：

#### 参数：

#### 返回值：

指示字母是否已填充的布尔值。

## 析构函数

### ~Thermodynamics()

#### 摘要：

#### 详细：

#### 参数：

#### 返回值：

### bool isrna

#### 摘要：

跟踪序列是RNA还是DNA。

#### 详细：

#### 参数：

#### 返回值：

# protected

### virtual void CopyThermo(const Thermodynamics& copy)

#### 摘要：

从Thermodynamics类的实例复制热力学参数。

#### 详细：

通常不需要这样做，因为函数会自动从磁盘填充参数。 但是，这在执行大量计算时很有用，因为参数只能从磁盘读取一次。<br> 请注意，必须使用“正确的” ISRNA值和正确的温度初始化源热力学类。<br> 如果GetErrorMessage（）或GetErrorMessageString（）可以解析没有错误和非零错误，则返回0。

#### 参数：

*\param* *thermo* 是指向Thermodynamics类的指针。 那一定已经调用了ReadThermodynamics（）函数。

**重要性** 除非可能在派生类的构造函数中使用，否则不应使用CopyThermo。<br> 请改用Thermodyanamic（Thermodynamic *复制）构造函数。<br> 原因：由于扩展了字母，数据表已加载到RNA类构造函数中，因此复制数据表也必须在构造函数中进行，以避免首先读取一个（可能不同的）数据表。

#### 返回值：

### datatable *data 

#### 摘要：

用于存储热力学参数的类。<br>

#### 详细：

 C字符串都将接收以路径名datapath开头的文件名。<br> isRNA指示是否正在读取DNA或RNA参数（true = RNA，false = DNA）。<br> isEnthlpy指示是否正在读取焓参数（true =焓，false = 37摄氏度下的自由能）。<br> 跟踪是否已读取参数文件。<br> 

#### 参数：

#### 返回值：

### datatable *enthalpy

#### 摘要：

用于存储焓参数的类。

#### 详细：

#### 参数：

#### 返回值：

### bool copied

#### 摘要：

数据表是否从另一个热力学类复制而来。 （在这种情况下，此类不应删除它。所有者应删除它。）<br>

#### 详细：

#### 参数：

#### 返回值：

### double nominal_temperature

#### 摘要：

 所需的折叠温度，单位为K（默认为310.15）<br> 请注意，如果在一个或多个其他Thermodynamics对象之间共享数据并且另一个已修改数据，则ACTUAL数据表的温度可能会与此温度不同。<br> 因此，如果已加载数据表，则应从this-> Set Temperature（）（在内部查询数据对象）中获取“ TRUE”温度

#### 详细：

#### 参数：

#### 返回值：

### string nominal_alphabetName

#### 摘要：

要打开的扩展字母的名称。

#### 详细：

请注意，如果在一个或多个其他数据之间共享数据，则ACTUAL数据表的字母可能会与此字母不同。<br> 因此，如果已加载数据表，则应从this-> GetAlphabetName（）（内部查询数据对象）中获取“ TRUE”字母

#### 参数：

#### 返回值：

### bool skipThermoTables

#### 摘要：

如果为true，则仅会加载字母。 能量表被跳过。 仅当程序不使用能量时（例如ct2dot等）才应将其设置为true

#### 详细：

#### 参数：

#### 返回值：
