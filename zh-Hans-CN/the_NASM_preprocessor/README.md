第四章 NASM 预处理器

NASM 拥有一个强大的宏处理器，它支持条件汇编，多级文件包含，两种形式的宏(单行的与多行的)，还有为更强大的宏能力而设置的`context stack`机制预处理指令都是以一个`%`打头。

预处理器把所有以反斜杠(\)结尾的连续行合并为一行，比如:

```nasm
       %define THIS_VERY_LONG_MACRO_NAME_IS_DEFINED_TO \ 
               THIS_VALUE
```

会像是单独一行那样正常工作。
