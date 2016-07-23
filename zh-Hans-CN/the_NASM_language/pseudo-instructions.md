3.2 伪指令。
======

伪指令是一些并不是真正的 x86 机器指令，但还是被用在了 instruction 域中的指令，因为使用它们可以带来很大的方便。

当前的伪指令有`DB`，`DW`，`DD`，`DQ`和`DT`，它们对应的未初始化指令是`RESB`，`RESW`，`RESD`，`RESQ`和`REST`，`INCBIN`命令，`EQU`命令和`TIEMS`前缀。

## 3.2.1 `DB`一类的伪指令: 声明已初始化的数据。

在 NASM 中，`DB`， `DW`， `DD`， `DQ`和`DT`经常被用来在输出文件中声明已初始化的数据，你可以多种方式使用它们:

```nasm

             db    0x55                ; just the byte 0x55 
             db    0x55,0x56,0x57      ; three bytes in succession 
             db    'a',0x55            ; character constants are OK 
             db    'hello',13,10,'$'   ; so are string constants 
             dw    0x1234              ; 0x34 0x12 
             dw    'a'                 ; 0x61 0x00 (it's just a number) 
             dw    'ab'                ; 0x61 0x62 (character constant) 
             dw    'abc'               ; 0x61 0x62 0x63 0x00 (string) 
             dd    0x12345678          ; 0x78 0x56 0x34 0x12 
             dd    1.234567e20         ; floating-point constant 
             dq    1.234567e20         ; double-precision float 
             dt    1.234567e20         ; extended-precision float

```

`DQ`和`DT`不接受数值常数或字符串常数作为操作数。

## 3.2.2 `RESB`类的伪指令: 声明未初始化的数据。

`RESB`， `RESW`， `RESD`， `RESQ` and `REST`被设计用在模块的 BSS 段中:它们声明未初始化的存储空间。

每一个带有单个操作数，用来表明字节数，字数，或双字数或其他的需要保留单位。

就像在 2.2.7 中所描述的，NASM 不支持 MASM/TASM 的扣留未初始化空间的语法`DW ?`或类似的东西:现在我们所描述的正是 NASM 自己的方式。

`RESB`类伪指令的操作数是有严格的语法的，参阅 3.8。

比如:

```nasm

       buffer:         resb    64              ; reserve 64 bytes 
       wordvar:        resw    1               ; reserve a word 
       realarray       resq    10              ; array of ten reals

```

## 3.2.3 `INCBIN`:包含其他二进制文件。

`INCBIN`是从老的 Amiga 汇编器 DevPac 中借过来的:它将一个二进制文件逐字逐句地包含到输出文件中。

这能很方便地在一个游戏可执行文件中包含中图像或声音数据。

它可以以下三种形式的任何一种使用:

```nasm
           incbin  "file.dat"             ; include the whole file 
           incbin  "file.dat",1024        ; skip the first 1024 bytes 
           incbin  "file.dat",1024,512    ; skip the first 1024, and 
                                          ; actually include at most 512
```

## 3.2.4 `EQU`: 定义常数。

`EQU`定义一个符号，代表一个常量值:当使用`EQU`时，源文件行上必须包含一个 label。

`EQU`的行为就是把给出的 label 的名字定义成它的操作数(唯一)的值。

定义是不可更改的，比如:

```nasm
       message         db      'hello, world' 
       msglen          equ     $-message
```

把`msglen`定义成了常量 12，`msglen`不能再被重定义。

这也不是一个预自理定义:`msglen`的值只被计算一次，计算中使用到了`$`(参阅 3.5)在此时的含义。

注意`EQU`的操作数也是一个严格语法的表达式。(参阅 3.8)

## 3.2.5 `TIMES`: 重复指令或数据。

前缀`TIMES`导致指令被汇编多次。

它在某种程序上是 NASM 的与 MASM 兼容汇编器的`DUP`语法的等价物。

你可以这样写:

```nasm
       zerobuf:        times 64 db 0
```

或类似的东西，但`TEIMES`的能力远不止于此。

`TIMES`的参数不仅仅是一个数值常数，还有数值表达式，所以你可以这样做:

```nasm
       buffer: db      'hello, world' 
               times 64-$+buffer db ' '
```

它可以把`buffer`的长度精确地定义为 64 字节，`TIMES`可以被用在一般地指令上，所以你可像这要编写不展开的循环:

```nasm
               times 100 movsb
```

注意在`times 100 resb 1`跟`resb 100`之间并没有显著的区别，除了后者在汇编时会快上一百倍。

就像`EQU`，`RESB`它们一样， `TIMES`的操作数也是严格语法的表达式。(见 3.8)

注意`TIMES`不可以被用在宏上:原因是`TIMES`在宏被分析后再被处理，它允许`TIMES`的参数包含像上面的`64-$+buffer`这样的表达式。

要重复多于一行的代码，或者一个宏，使用预处理指令`%rep`。