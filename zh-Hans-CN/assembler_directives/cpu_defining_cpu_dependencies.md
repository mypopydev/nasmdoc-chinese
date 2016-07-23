5.7 `CPU`: 定义 CPU 相关。
======

`CPU`指令限制只能运行特定 CPU 类型上的指令。

选项如下:

*  `CPU 8086` 只汇编 8086 的指令集。
*  `CPU 186` 汇编 80186 及其以下的指令集。
*  `CPU 286` 汇编 80286 及其以下的指令集。
*  `CPU 386` 汇编 80386 及其以下的指令集。
*  `CPU 486` 486 指令集。
*  `CPU 586` Pentium 指令集。
*  `CPU PENTIUM` 同 586。
*  `CPU 686` P6 指令集。
*  `CPU PPRO` 同 686
*  `CPU P2` 同 686
*  `CPU P3` Pentium III and Katmai 指令集。
*  `CPU KATMAI` 同 P3
*  `CPU P4` Pentium 4 (Willamette)指令集
*  `CPU WILLAMETTE` 同 P4
*  `CPU IA64` IA64 CPU (x86 模式下)指令集所有选项都是大小写不敏感的，在指定 CPU 或更低一级 CPU 上的所有指令都会被选择。

缺省情况下，所有指令都是可用的。