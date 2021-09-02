---
title: ELF文件的结构
date: 2021-08-30 15:22:49
tags: 
- File
categories:
description:
---

#### 一、概述
代码是解决问题的工具。当通过编码方式解决一个问题时，通常有如下步骤：
1.定义问题
2.设计解决方案
3.编码实现方案
    3.1 编译源码，生成目标文件
    3.2 链接目标文件，生成可执行文件或库
4.测试程序

那么，什么是目标文件呢？

我们用IDE编写的代码是普通的文本文件，要想被机器识别，需要经过一些处理转换成二进制格式。
这个处理过程包括：
+ 预处理
+ 编译
+ 汇编
+ 链接

预处理过程由预处理器（Preprocessor）完成：
+ \#define宏定义展开
+ 处理条件编译指令
+ 处理\#include指令
+ 删除注释
+ 添加行号
+ 保留\#pragma指令
编译过程由编译器（Compiler）完成：转换成汇编代码
+ 词法分析、语法分析、语义分析和优化
汇编过程由汇编器（Assembler）完成：转换成二进制的机器码
+ 汇编码到机器码的翻译
链接过程由链接器（Linker）完成：将上一步生成的目标文件和其他目标文件一起链接生成可执行文件

<center>
    <img src="../images/GCC_CompilationProcess.png" width="400"/>
</center>

综上可知，目标文件是程序未链接前的一个中间文件。

#### 二、文件结构解析

目标文件其实是ELF文件的一种，常见的ELF文件还有.so和.oat文件。ELF文件存在两种观察角度，分别是编译角度和运行角度。两个角度看到的结构略有差异。如图：

<center>
    <img src="../images/elf_format_overview.png" width="400"/>
</center>

从图中可以得到这样一些信息：Program Header Table对于编译视图是可选的，Section Header Table对于执行视图是可选的。
编译视图是以section为组织单位，执行视图是以segment为组织单位，下面介绍一些重要的结构。

##### ELF Header

```
#define EI_NIDENT 16
typedef struct {
    unsigned char e_ident[EI_NIDENT];
    //此elf文件的类型
    Elf32_Half e_type;//ET_NONE An unknown type.
                        ET_REL A relocatable file.
                        ET_EXEC An executable file.
                        ET_DYN A shared object.
                        ET_CORE A core file.
    Elf32_Half e_machine;
    Elf32_Word e_version;
    Elf32_Addr e_entry;
    Elf32_Off e_phoff;
//section header table在此文件中的偏移量
    Elf32_Off e_shoff;
    Elf32_Word e_flags;
    Elf32_Half e_ehsize;
//program-header-table数组中一个元素的大小
    Elf32_Half e_phentsize;
//program-header-table数组元素的个数
    Elf32_Half e_phnum;
    Elf32_Half e_shentsize;
//section header table包含的元素个数
    Elf32_Half e_shnum;
//section header table每个元素的字节大小
    Elf32_Half e_shstrndx;
 } Elf32_Ehdr;
```



##### Program Header Table
用于ELF文件装载进内存时进行分块

```
typedef struct {
//此元素描述的是哪种segment，以及如何解析
   uint32_t   p_type;
//segment首字节在文件中的偏移值
   Elf32_Off  p_offset;
//应该加载到内存中的哪个虚拟地址中
   Elf32_Addr p_vaddr;
//相关的物理地址，BSD中为0
   Elf32_Addr p_paddr;
//文件的字节数
   uint32_t   p_filesz;
//加载到内存中的字节数
   uint32_t   p_memsz;
//segment类型标志位
//PF_X   An executable segment.
//PF_W   A writable segment.
//PF_R   A readable segment.
   uint32_t   p_flags;
//
   uint32_t   p_align;
} Elf32_Phdr;
```


##### Section Header Table

用于定位文件中所有的section，它是一个元素为Elf32_Shdr的数组结构。
```
typedef struct
{
	Elf32_Word sh_name;      /* Section name (string tbl index)   */
	Elf32_Word sh_type;      /* Section type                      */
	Elf32_Word sh_flags;     /* Section flags                     */
	Elf32_Addr sh_addr;      /* Section virtual addr at execution */
	Elf32_Off sh_offset;     /* Section file offset               */
	Elf32_Word sh_size;      /* Section size in bytes             */
	Elf32_Word sh_link;      /* Link to another section           */
	Elf32_Word sh_info;      /* Additional section information    */
	Elf32_Word sh_addralign; /* Section alignment                 */
	Elf32_Word sh_entsize;   /* Entry size if section holds table */
} Elf32_Shdr;
```

##### 字符串表和符号表（String and symbol tables）
存放scetion的名字，以及符号信息。其他结构通过index访问此结构中的字符串信息。符号表保存了用于定位符号引用的信息。它是以下结构的数组：

<center>
    <img src="../images/elf_string_table.jpeg" width="500"/>
</center>

```
typedef struct {
       uint32_t      st_name;
       Elf32_Addr    st_value;
       uint32_t      st_size;
       unsigned char st_info;
       unsigned char st_other;
       uint16_t      st_shndx;
} Elf32_Sym;
```


##### Relocation entries (Rel & Rela)
重定位是连接符号引用和符号定义的过程。重定位文件必须包含如何修改section内容的信息，来让执行文件或共享目标文件正确的调用外部函数。

```
typedef struct {
        Elf32_Addr      r_offset;
        Elf32_Word      r_info;
} Elf32_Rel;
 
typedef struct {
        Elf32_Addr      r_offset;
        Elf32_Word      r_info;
        Elf32_Sword     r_addend;
} Elf32_Rela;
```
r_offset：
对于重定位文件来说，此值表示从section的开始到被修改值在section中的偏移量。
对于执行文件和共享库文件来说，此值表示需要被重定位的虚拟地址。

r_info：
符号表的index和重定位的类型。

r_addend：
用于计算重定位字段的值。

#### 三、参考链接

[Linux manual page](https://www.man7.org/linux/man-pages/man5/elf.5.html)
[The structure of an ARM ELF image](https://www.codetd.com/article/489055)
[Compiling, Linking and Building](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html)
