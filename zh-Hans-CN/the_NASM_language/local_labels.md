3.9 本地 Labels
======

NASM 对于那些以一个句点开始的符号会作特殊处理，一个以单个句点开始的Label 会被处理成本地 label， 这意味着它会跟前面一个非本地 label 相关联.比如:

```nasm
       label1  ; some code 
       
       .loop 
               ; some more code 
       
               jne     .loop 
               ret 
       
       label2  ; some code 
       
       .loop 
               ; some more code 
       
               jne     .loop 
               ret
```

上面的代码片断中，每一个`JNE`指令跳至离它较近的前面的一行上，因为`.loop`的两个定义通过与它们前面的非本地 Label 相关联而被分离开来了。

对于本地 Label 的处理方式是从老的 Amiga 汇编器 DevPac 中借鉴过来的;

尽管如此，NASM 提供了进一步的性能，允许从另一段代码中调用本地 labels。

这是通过在本地 label 的前面加上非本地 label 前缀实现的:第一个.loop 实际上被定义为`label1.loop`，而第二个符号被记作`label2.loop`。

所以你确实需要的话你可写:

```nasm
       label3  ; some more code 
               ; and some more 
       
               jmp label1.loop
```

有时，这是很有用的(比如在使用宏的时候)，可以定义一个 label，它可以在任何地方被引用，但它不会对常规的本地 label 机制产生干扰。

这样的label 不能是非本地 label，因为非本地 label 会对本地 labels 的重复定义与引用产生干扰;

也不能是本地的，因为这样定义的宏就不能知道 label 的全称了。

所以 NASM 引进了第三类 label，它只在宏定义中有用:如果一个 label以一个前缀`..@`开始，它不会对本地 label 产生干扰，所以，你可以写:

```nasm
       label1:                         ; a non-local label 
       .local:                         ; this is really label1.local 
       ..@foo:                         ; this is a special symbol 
       label2:                         ; another non-local label 
       .local:                         ; this is really label2.local 
       
               jmp     ..@foo          ; this will jump three lines up
```

NASM 还能定义其他的特殊符号，比如以两个句点开始的符号，比如`..start`被用来指定`.obj`输出文件的执行入口。(参阅 6.2.6)

第四章 NASM 预处理器

NASM 拥有一个强大的宏处理器，它支持条件汇编，多级文件包含，两种形式的宏(单行的与多行的)，还有为更强大的宏能力而设置的`context stack`机制预处理指令都是以一个`%`打头。

预处理器把所有以反斜杠(\)结尾的连续行合并为一行，比如:

```nasm
       %define THIS_VERY_LONG_MACRO_NAME_IS_DEFINED_TO \ 
               THIS_VALUE
```

会像是单独一行那样正常工作。