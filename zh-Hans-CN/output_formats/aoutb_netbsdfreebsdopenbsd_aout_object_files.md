6.7 `aoutb`: NetBSD/FreeBSD/OpenBSD `a.out`目标文件.
======

`aoutb`格式产生在 BSD unix，NetBSD，FreeBSD，OpenBSD 系统上使用的`a.out`目标文件. 作为一种简单的目标文件，这种格式跟`aout`除了开头四字节的魔数不一样，其他完全相同.但是，`aoutb`格式支持跟 elf 格式一样的地址无关代码，所以你可以使用它来写`BSD`共享库.`aoutb`提供的缺省文件扩展名是`.o`.`aoutb`不支持特殊的操作符，没有特殊的符号，只有三个殊殊的段名`.text`，`.data`和`.bss`.但是，它象 elf 一样支持`WRT`的使用，这是为了提供地址无关的代码重定位类型.关于这部分的完整文档，请参阅 6.5.2`aoutb`也支持跟`elf`同样的对于`GLOBAL`的扩展:详细信息请参阅 6.5.3.