5.3 `ABSOLUTE`: 定义绝对 labels。
======

`ABSOLUTE`操作符可以被认为是`SECTION`的另一种形式:它会让接下来的代码不在任何的物理段中，而是在一个从给定地址开始的假想段中。

在这种模式中，你唯一能使用的指令是`RESB`类指令。

`ABSOLUTE`可以象下面这样使用:

```nasm
       absolute 0x1A 
       
           kbuf_chr    resw    1 
           kbuf_free   resw    1 
           kbuf        resw    16
```

这个例子描述了一个关于在段地址 0x40 处的 PC BIOS 数据域的段，上面的代码把`kbuf_chr`定义在 0x1A 处，`kbuf_free`定义在地址 0x1C 处，`kbuf`定义在地址0x1E。

就像`SECTION`一样，用户级的`ABSOLUTE`在执行时会重定义`__SECT__`宏。

`STRUC`和`ENDSTRUC`被定义成使用`ABSOLUTE`的宏(同时也使用了`__SECT__`)`ABSOLUTE`不一定需要带有一个绝对常量作为参数:它也可以带有一个表达式(实际上是一个临界表达式，参阅 3.8)，表达式的值可以是在一个段中。

比如，一个 TSR 程序可以在用它重用它的设置代码所占的空间:

```nasm
               org     100h               ; it's a .COM program 
       
               jmp     setup              ; setup code comes last 
       
               ; the resident part of the TSR goes here 
       setup: 
               ; now write the code that installs the TSR here 
       
       absolute setup 
       
       runtimevar1     resw    1 
       runtimevar2     resd    20 
       
       tsr_end:

```

这会在 setup 段的开始处定义一些变量，所以，在 setup 运行完后，它所占用的内存空间可以被作为 TSR 的数据存储空莘而得到重用。

符号`tsr_end`可以用来计算 TSR程序所需占用空间的大小。