---
title: 类文件结构
date: 2022-02-27
tags: jvm
categories: Java学习
cover: /img/cover/jvm.jpg
---



# Class类文件的结构

Class文件是一组以8个字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在文件之中，中间没有添加任何分隔符，这使得整个Class文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在。当遇到需要占用8个字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8个字节进行存储。

根据《Java虚拟机规范》的规定，Class文件格式采用一种类似于C语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型：“无符号数“和”表“。

- 无符号数属于基本的数据类型，以u1，u2，u4，u8来分别代表1个字节，2个字节，4个字节，8个字节的无符号数，无符号数可以用来描述数字，索引引用，数量值或者按照UTF-8编码构成字符串值。
- 表是由多个无符号数或者其他表作为数据项构成的复合数据类型，为了便于区分，所有表的命名都习惯性地以`_info`结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质也可以视作是一张表。

| 类型           | 名称                | 数量                  |
| -------------- | ------------------- | --------------------- |
| u4             | magic               | 1                     |
| u2             | minor_version       | 1                     |
| u2             | major_version       | 1                     |
| u2             | constant_pool_count | 1                     |
| cp_info        | constant_pool       | constant_pool_count-1 |
| u2             | access_flags        | 1                     |
| u2             | this_class          | 1                     |
| u2             | super_class         | 1                     |
| u2             | interfaces_count    | 1                     |
| u2             | interfaces          | interfaces_count      |
| u2             | fields_count        | 1                     |
| field_info     | fields              | fields_count          |
| u2             | methods_count       | 1                     |
| method_info    | methods             | methods_count         |
| u2             | attributes_count    | 1                     |
| attribute_info | attributes          | attributes_count      |

## 1 魔数与Class文件的版本

每个Class文件开头四位被称为魔数`Magic Number`，它的唯一目的是确认这个文件是否为一个能被虚拟机接受的Class文件。不仅是Class文件，很多文件格式标准中都有用魔数来进行身份识别的习惯，譬如图片格式，如GIF或JPEG等文件头中都有存有魔数。使用魔数而不是扩展名来进行识别主要是基于安全，因为文件扩展名可以随意改动。Class文件的魔数值为`0xCAFEBAE`。

紧接着魔数的4个字节存储的是Class文件的版本号：第5和第6是次版本号`Minor Version`，第7和第8字节是主版本号`Major Version`。Java的版本号是从45开始的，JDK1.1之后每个JDK大版本发布主版本号向上加1，高版本的JDK能向下兼容以前版本的Class文件，但是不能运行以后版本的Class文件，《Java虚拟机规范》在Class文件校验部分明确要求了即使文件格式并未发生变化，虚拟机也必须拒绝执行超过其版本号到Class文件。

## 2 常量池

紧接着主，次版本号之后的事常量池入口，常量池可以比喻为Class文件里的资源仓库，它是Class文件结构中与其他项目关联最多的数据，通常也是占用Class文件空间最大的数据项目之一，另外，它还是在class文件中第一个出现的表类型数据项目。

由于常量池中常量数量是不固定的，所以在常量池的入口需要放置一项u2类型的数据，代表常量池容量计数器`constant_pool_count`。与Java语言习惯不同，这个容量计数从1而不是从0开始。设计者将第0项常量空出来是有特殊考虑的，这样做的目的在于，如果后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义。可以把索引设置为0来表示。Class文件结构中只有常量池的容量计数器是从1开始，对于其他集合类型，包括接口索引集合，字段表集合，方法表集合等的容量计数都与一般习惯相同，是从0开始。

常量池中主要存放两大类常量：字面量`Literal`和符号引用`Symbolic References`。字面量比较接近Java语言层面的常量概念，如文本字符串，被声明为final的常量值等。而符号引用则属于编译原理方面的概念，主要包括下面几类常量：

- 被模块导出或者开放的包`Package`
- 类和接口的全限定名`Full Qualified Name`
- 字段的名称和描述符`Descriptor`
- 方法的名称和描述符
- 方法句柄和方法类型`Method Handle,Method Type,Invoke Dynamic`
- 动态调用点和动态常量`Dynamically-Computed Call Site,Dynamically-Computed Constant`

在Class文件中不会保存各个方法，字段最终在内存中的布局信息，这些字段，方法的符号引用不经过虚拟机在运行期转换的话是无法得到真正的内存入口地址，也就无法直接被虚拟机使用的。当虚拟机做类加载时，将会从常量池获取对应的符号引用，再在类创建时或者运行时解析，翻译到具体的内存地址中。

常量池中每一项常量都是一个表，截止JDK13，常量表中分别有17种不同类型的常量。这些表都有一个共同的特点，表结构起始的第一位是个u1类型的标志位，代表着当前常量属于哪种常量类型。

