6.5 `elf`: 可执行可连接格式目标文件.
======

`elf`输出格式产生`ELF32`(可执行可连接格式)目标文件，这种格式用在 Linux，Unix System V 中，包括 Solaris x86， UnixWare 和 SCO Unix. `elf`提供一个缺省的输出文件扩展名`.o`.

## 6.5.1 `elf`对`SECTION`操作符的扩展.

象`obj`格式一样，`elf`允许你在`SECTION`操作符行上指定附加的信息，以控制你声明的段的类型与属性.对于标准的段名`.text`，`.data`，`.bss`，NASM 都会产生缺省的类型与属性.但还是可以通过一些限定符与重新指定.可用的限定符如下:

*  `alloc`定义一个段，在程序运行时，这个段必须被载入内存中，`noalloc`正好相反，比如信息段，或注释段.
*  `exec`把段定义为在程序运行的时候必须有执行权限.`noexec`正好相反.
*  `write`把段定义为在程序运行时必须可写，`nowrite`正好相反.
*  `progbits`把段定义为在目标文件中必须有实际的内容，比如象普通的代码段与数据段，`nobits`正好相反，比如`bss`段.
*  `align=`跟上一个数字，给出段的对齐请求.如果你没有指定上述的限定符信息，NASM 缺省指定的如下:

```nasm
       section .text    progbits  alloc  exec    nowrite  align=16 
       section .rodata  progbits  alloc  noexec  nowrite  align=4 
       section .data    progbits  alloc  noexec  write    align=4 
       section .bss     nobits    alloc  noexec  write    align=4 
       section other    progbits  alloc  noexec  nowrite  align=1
```

(任何不在上述列举范围内的段，在缺省状况下，都被作为`other`段看待).

## 6.5.2 地址无关代码: `elf`特定的符号和 `WRT`

`ELF`规范含有足够的特性以允许写地址无关(PIC)的代码，这可以让 ELF非常方便地共享库.尽管如此，这也意味着 NASM 如果想要成为一个能够写 PIC 的汇编器的话，必须能够在 ELF目标文件中产生各种奇怪的重定位信息，因为`ELF`不支持基于段的地址引用，`WRT`操作符不象它的常规方式那样被使用，所以，NASM 的`elf`输出格式中，对于`WRT`有特殊的使用目的，叫做:PIC 相关的重定位类型.`elf`定义五个特殊的符号，它们可以被放在`WRT`操作符的右边用来实现 PIC重定位类型.它们是`..gotpc`， `..gotoff`， `..got`， `..plt` and `..sym`.它们的功能简要介绍如下:

*  使用`wrt ..gotpc`来引用以 global offset table 为基址的符号会得到当前段的起始地址到 globaloffset table 的距离.(`_GLOBAL_OFFSET_TABLE_`是引用 GOT的标准符号名).所以你需要在返回结果前面加上`$$`来得到 GOT 的真实地址.
*  用`wrt ..gotoff`来得到你的某一个段中的一个地址实际上得到从 GOT的的起始地址到你指定的地址之间的距离，所以这个值再加上 GOT 的地址为得到你需要的那个真实地址.
*  使用`wrt ..got`来得到一个外部符号或全局符号会让连接器在含有这个符号的地址的 GOT中建立一个入口，这个引用会给出从 GOT 的起始地址到这个入口的一个距离;所以你可以加上 GOT的地址，然后从得到的地址处载入，就会得到这个符号的真实地址.
*  使用`wrt ..plt`来引用一个过程名会让连接器建立一个过程连接表入口，这个引用会给出 PLT入口的地址.你可以在上下文使用这个引用，它会产生PC 相关的重定位信息，所以，ELF 包含引用 PLT入口的非重定位类型
*  略在 8.2 章中会有一个更详细的关于如何使用这些重定位类型写共享库的介绍


## 6.5.3 `elf`对`GLOBAL`操作符的扩展.

`ELF`目标文件可以包含关于一个全局符号的很多信息，不仅仅是一个地址:他可以包含符号的 size，和它的类型.这不仅仅是为了调试的方便，而且在写共享库程序的时候，这确实是非常有用的.所以，NASM 支持一些关于`GLOBAL`操作符的扩展，允许你指定这些特性.你可以把一个全局符号指定为一个函数或一个数据对象，这是通过在名字后面加上一个冒号跟上`function`或`data`实现的.(`object`可以用来代替`data`)比如:

```nasm
       global   hashlookup:function, hashtable:data
```

把全局符号`hashlookup`指定为一个函数，把`hashtable`指定为一个数据对象.你也可以指定跟这个符号关联的数据的 size，可以一个数值表达式(它可以包含labels，甚至前向引用)跟在类型后面，比如:

```nasm
       global  hashtable:data (hashtable.end - hashtable) 
       
       hashtable: 
               db this,that,theother  ; some data here 
       .end:
```

这让 NASM 自动计算表的长度，然后把信息放进`ELF`的符号表中.声明全局符号的类型和 size 在写共享库代码的时候是必须的，关于这方面的更多信息，参阅 8.2.4.

## 6.5.4 `elf`对`COMMON`操作符的扩展.

`ELF`也允许你指定通用变量的对齐请求.这是通过在通用变量的名字和 size 的后面加上一个以冒号分隔的数字来实现的，比如，一个 doubleword 的数组以4byte 对齐比较好:

```nasm
       common  dwordarray 128:4
```

这把 array 总的 size 声明为 128bytes，并确定它对齐到 4byte 边界.

## 6.5.5 16 位代码和 ELF

`ELF32`规格不提供关于 8 位和 16 位值的重定位，但 GNU 的连接器`ld`把这作为一个扩展加进去了.NASM 可以产生 GNU 兼容的重定位，允许 16 位代码被`ld`以`ELF`格式进行连接.如果 NASM 使用了选项`-w+gnu-elf-extensions`，如果一个重定位被产生的话，会有一条警告信息.
