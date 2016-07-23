4.8 标准宏。
======

NASM 定义了一套标准宏，当开始处理源文件时，这些宏都已经被定义了。

如果你真的希望一个程序在执行前没有预定义的宏存在，你可以使用`%clear`操作符清空预处理器的一切。

大多数用户级的操作符(第五章)是作为宏来运行的，这些宏进一步调用原始的操作符;

这些在第五章介绍。

剩余的标准宏在这里进行描述。


## 4.8.1 `__NASM_MAJOR__`， `__NASM_MINOR__`， `__NASM_SUBMINOR__`和`___NASM_PATCHLEVEL__`: NASM 版本宏。

单行宏`__NASM_MAJOR__`， `__NASM_MINOR__`，`__NASM_SUBMINOR__`和`___NASM_PATCHLEVEL__`被展开成当前使用的 NASM 的主版本号，次版本号，子次版本号和补丁级。

所在，在 NASM 0.98.32p1 版本中，`__NASM_MAJOR__`被展开成 0，`__NASM_MINOR__`被展开成 98，`__NASM_SUBMINOR__`被展开成32，`___NASM_PATCHLEVEL__`被定义为 1。

## 4.8.2 `__NASM_VERSION_ID__`: NASM 版本 ID。

单行宏`__NASM_VERSION_ID__`被展开成双字整型数，代表当前使用的版本的 NASM 的全版本数。

这个值等于把`__NASM_MAJOR__`，`__NASM_MINOR__`，`__NASM_SUBMINOR__`和`___NASM_PATCHLEVEL__`连结起来产生一个单个的双字长整型数。

所以，对于 0.98.32p1，返回值会等于

 
 ```nasm
       dd      0x00622001
 ```

 或者

 ```nasm
       db      1,32,98,0
 ```
 
 注意，上面两行代码产生的是完全相同的代码，第二行只是用来指出内存中存在的各个值之间的顺序。

## 4.8.3 `__NASM_VER__`: NASM 版本字符串。

单行宏`__NASM_VER__`被展开成一个字符串，它定义了当前使用的 NASM 的版本号。

所以，在 NASM0.98.32 下:

```nasm
        db      __NASM_VER__
```

会被展开成:

```nasm
       db      "0.98.32"
```

## 4.8.4 `__FILE__` and `__LINE__`: 文件名和行号。

就像 C 的预处理器，NASM 允许用户找到包含有当前指令的文件的文件名和行数。

宏`__FILE__`展开成一个字符串常量，该常量给出当前输入文件的文件名(如果含有`%include`操作符，这个值会在汇编的过程中改变)，而`__LINE__`会被展开成一个数值常量，给出在输入文件中的当前行的行号。

这些宏可以使用在宏中，以查看调试信息，当在一个宏定义中包含宏`__LINE__`时(不管 是单行还是多行)，会返回宏调用，而不是宏定义处的行号。

这可以用来确定是否在一段代码中发生了程序崩溃。

比如，某人可以编写一个子过程`stillhere`，它通过`EAX`传递一个行号，然后输出一些信息，比如:"line 155: still here`。

你可以这样编写宏:

```nasm
       %macro  notdeadyet 0 
       
               push    eax 
               mov     eax,__LINE__ 
               call    stillhere 
               pop     eax 
       
       %endmacro
```

然后，在你的代码中插入宏调用，直到你发现发生错误的代码为止。

## 4.8.5 `STRUC` and `ENDSTRUC`: 声明一个结构体数据类型。

在 NASM 的内部，没有真正意义上的定义结构体数据类型的机制;

 取代它的是，预处理器的功能相当强大，可以把结构体数据类型以一套宏的形式来运行。

宏 `STRUCT` 和`ENDSTRUC`是用来定义一个结构体数据类型的。

`STRUCT`带有一个参数，它是结构体的名字。

这个名字代表结构体本身，它在结构体内的偏移地址为零，名字加上一个_size 后缀组成的符号，用一个`EQU`给它赋上结构体的大小。

一旦`STRUC`被执行，你就开始在定义一个结构体，你可以用`RESB`类伪指令定义结构体的域，然后使用`ENDSTRUC`来结束定义。

比如，定义一个叫做`mytype`的结构体，包含一个 longword，一个 word，一个byte，和一个字符串，你可以这样写代码:

```nasm
       struc   mytype 
       
         mt_long:      resd    1 
         mt_word:      resw    1 
         mt_byte:      resb    1 
         mt_str:       resb    32 
       
       endstruc
```

上面的代码定义了六个符号:`m_long`在地置 0(从结构体`mytype`开头开始到这个 longword 域的偏移)，`mt_word`在地置 4， `mt_byte`6， `mt_str`7，`mytype_size`是 39，而`mytype` 自己在地置 0之所以要把结构体的名字定义在地址零处，是因为要让结构体可以使用本地labels 机制的缘故:如果你想要在多个结构体中使用具有同样名字的成员，你可以把上面的结构体定义成这个样子:

```nasm
       struc mytype 
       
         .long:        resd    1 
         .word:        resw    1 
         .byte:        resb    1 
         .str:         resb    32 
       
       endstruc
```

在这个定义中，把结构体域的偏移值定义成了:`mytype.long`，`mytype.word`， `mytype.byte` and `mytype.str`.NASM 因此而没有内部的结构体支持，也不支持以句点形式引用结构体中的成员，所以代码`mov ax， [mystruc.mt_word]`是非法的，`mt_word`是一个常数，就像其它类型的常数一样，所以，正确的语法应该是`mov ax，[mystruc+mt_word]`或者`mov ax，[mystruc+mytype.word]`.

## 4.8.6 `ISTRUC`， `AT` and `IEND`: 声明结构体的一个实例。

定义了一个结构体类型以后，你下一步要做的事情往往就是在你的数据段中声明一个结构体的实例。

NASM 通过使用`ISTRUC`机制提供一种非常简单的方式。

在程序中声明一个`mytype`结构体，你可以象下面这样写代码:

```nasm
       mystruc: 
           istruc mytype 
       
               at mt_long, dd      123456 
               at mt_word, dw      1024 
               at mt_byte, db      'x' 
               at mt_str,  db      'hello, world', 13, 10, 0 
       
           iend
```

`AT`宏的功能是通过使用`TIMES`前缀把偏移位置定位到正确的结构体域上，然后，声明一个特定的数据。

所以，结构体域必须以在结构体定义中相同的顺序被声明。

如果为结构体的域赋值要多于一行，那接下的内容可直接跟在`AT`行后面，比如:

```nasm
        at mt_str,  db      123,134,145,156,167,178,189 
                    db      190,100,0
```

按个人的喜好不同，你也可以不在`AT`行上写数据，而直接在第二行开始写数据域:

```nasm
        at mt_str 
                db      'hello, world' 
                db      13,10,0
```

## 4.8.7 `ALIGN` and `ALIGNB`: 数据对齐

宏`ALIGN`和`ALIGNB`提供一种便捷的方式来进行数据或代码的在字，双字，段或其他边界上的对齐(有些汇编器把这两个宏叫做`EVEN`)，有关这两个宏的语法是:

```nasm
        align   4               ; align on 4-byte boundary 
        align   16              ; align on 16-byte boundary 
        align   8,db 0          ; pad with 0s rather than NOPs 
        align   4,resb 1        ; align to 4 in the BSS 
        alignb  4               ; equivalent to previous line
```

这两个个参数都要求它们的第一个参数是 2 的幂;

它们都会计算需要多少字节来来存储当前段，当然这个字节数必须向上对齐到一个 2 的幂值。

然后用它们的第二个参数来执行`TIMES`前缀进行对齐。

如果第二个参数没有被指定，那`ALIGN`的缺省值就是`NOP`，而`ALIGNB`的缺省值就是`RESB 1`.当第二个参数被指定时，这两个宏是等效的。

通常，你可以在数据段与代码段中使用`ALIGN`，而在 BSS 段中使用`ALIGNB`，除非有特殊用途，一般你不需要第二个参数。

作为两个简单的宏，`ALIGN`与`ALIGNB`不执行错误检查:如果它们的第一个参数不是 2 的某次方，或它们的第二个参数大于一个字节的代码，他们都不会有警告信息，这两种情况下，它们都会执行错误。

`ALIGNB`(或者，`ALIGN`带上第二个参数`RESB 1`)可以用在结构体的定义中:

```nasm
       struc mytype2 
       
         mt_byte: 
               resb 1 
               alignb 2 
         mt_word: 
               resw 1 
               alignb 4 
         mt_long: 
               resd 1 
         mt_str: 
               resb 32 
       
       endstruc
```

这可以保证结构体的成员被对齐到跟结构体的基地址之间有一个正确的偏移值。

最后需要注意的是，`ALIGN`和`ALIGNB`都是以段的开始地址作为参考的，而不是整个可执行程序的地址空间。

如果你所在的段只能保证对齐到 4 字节的边界，那它会对齐对 16 字节的边界，这会造成浪费，另外，NASM 不会检测段的对齐特性是否可被`ALIGN`和`ALIGNB`使用。