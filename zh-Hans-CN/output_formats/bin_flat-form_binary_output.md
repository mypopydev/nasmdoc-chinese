6.1 `bin`: 纯二进制格式输出。
======

`bin`格式不产生目标文件:除了你编写的那些代码，它不在输出文件中产生任何东西。

这种纯二进制格式的文件可以用在 MS-DOS 中:`.COM`可执行文件和`.SYS`设备驱动程序就是纯二进制格式的。

纯二进制格式输出对于操作系统和引导程序开发也是很有用的。

`bin`格式支持多个段名。

关于 NASM 处理`bin`格式中的段的细节，请参阅 6.1.3。

使用`bin`格式会让 NASM 进入缺省的 16 位模式 (参阅 5.1)。

为了能在`bin`格式中使用 32 位代码，比如在操作系统的内核代码中。

你必须显式地使用`BITS 32`操作符。

`bin`没有缺省的输出文件扩展名:它只是把输入文件的扩展名去掉后作为输出文件的名字。

这样，NASM 在缺省模式下会把`binprog.asm`汇编成二进制文件`binprog`。

## 6.1.1 `ORG`: 二进制程序的起点位置。

`bin`格式提供一个额外的操作符，这在第五章已经给出`ORG`.`ORG`的功能是指定程序被载入内存时，它的起始地址。

比如，下面的代码会产生 longword: `0x00000104`:

```nasm
               org     0x100 
               dd      label 
       label:
```

跟 MASM 兼容汇编器提供的`ORG`操作符不同，它们允许你在目标文件中跳转，并覆盖掉你已经产生的代码，而 NASM 的`ORG`就象它的字面意思“起点”所表示的，它的功能就是为所有内部的地址引用增加一个段内偏移值;

它不允许 MASM 版本的`org`的任何其他功能。

## 6.1.2 `bin`对`SECTION`操作符的扩展。


`bin`输出格式扩展了`SECTION`(或者`SEGMENT`)操作符，允许你指定段的对齐请求。

这是通过在段定义行的后面加上`ALIGN`限定符实现的。

比如:

```nasm
       section .data   align=16
```

它切换到段`.data`，并指定它必须对齐到 16 字节边界。

`ALIGN`的参数指定了地址值的低位有多少位必须为零。

这个对齐值必须为2 的幂。


## 6.1.3 `Multisection` 支持 BIN 格式.

`bin`格式允许使用多个段，这些段以一些特定的规则进行排列。

*  任何在一个显式的`SECTION`操作符之前的代码都被缺省地加到`.text`段中。
*  如果`.text`段中没有给出`ORG`语句，它被缺省地赋为`ORG 0`。
*  显式地或隐式地含有`ORG`语句的段会以`ORG`指定的方式存放。代码前会填充 0，以在输出文件中满足 org 指定的偏移。
*  如果一个段内含有多个`ORG`语句，最后一条`ORG`语句会被运用到整个段中，不会影响段内多个部分以一定的顺序放到一起。
*  没有`ORG`的段会被放到有`ORG`的段的后面，然后，它们就以第一次声明时的顺序被存放。
*  `.data`段不像`.text`段和`.bss`段那样，它不遵循任何规则，
*  `.bss`段会被放在所有其他段的后面。
*  除非一个更高级别的对齐被指定，所有的段都被对齐到双字边界。
*  段之间不可以交迭。

##  6.1.4 Map files

       Map files can be generated in `-f bin' format by means of the
       `[map]' option. Map types of `all' (default), `brief', `sections',
       `segments', or `symbols' may be specified. Output may be directed to
       `stdout' (default), `stderr', or a specified file. E.g.
       `[map symbols myfile.map]'. No "user form" exists, the square
       brackets must be used.
