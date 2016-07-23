6.2 `obj`: 微软 OMF 目标文件
======

`obj`文件格式(因为历史的原因，NASM 叫它`obj`而不是`omf`)是 MASM 和TASM 可以产生的一种格式，它是标准的提供给 16 位的 DOS 链接器用来产生`.EXE`文件的格式。

它也是 OS/2 使用的格式。

`obj`提供一个缺省的输出文件扩展名`.obj`。

`obj`不是一个专门的 16 位格式，NASM 有一个完整的支持，可以有它的 32 位扩展。

32 位 obj 格式的文件是专门给 Borland 的 Win32 编译器使用的，这个编译器不使用微软的新的`win32`目标文件格式。

`obj`格式没有定义特定的段名字:你可以把你的段定义成任何你喜欢的名字。

一般的，obj 格式的文件中的段名如:`CODE`， `DATA`和`BSS`.如果你的源文件在显式的`SEGMENT`前包含有代码，NASM 会为你创建一个叫做`__NASMDEFSEG`的段以包含这些代码.当你在 obj 文件中定义了一个段，NASM 把段名定义为一个符号，所以你可以存取这个段的段地址。

比如:

```nasm
       segment data 
       
       dvar:   dw      1234 
       
       segment code 
       
       function: 
               mov     ax,data         ; get segment address of data 
               mov     ds,ax           ; and move it into DS 
               inc     word [dvar]     ; now this reference will work 
               ret
```

obj 格式中也可以使用`SEG`和`WRT`操作符，所以你可以象下面这样编写代码:

```nasm
       extern  foo 
       
             mov   ax,seg foo            ; get preferred segment of foo 
             mov   ds,ax 
             mov   ax,data               ; a different segment 
             mov   es,ax 
             mov   ax,[ds:foo]           ; this accesses `foo' 
             mov   [es:foo wrt data],bx  ; so does this
```

## 6.2.1 `obj` 对`SEGMENT`操作符的扩展。

obj 输出格式扩展了`SEGMENT`(或`SECTION`)操作符，允许你指定段的多个属性。

这是通过在段定义行的末尾添加额外的限定符来实现的，比如:

```nasm
       segment code private align=16
```

这定义了一个段`code`，但同时把它声明为一个私有段，同时，它所描述的这个部分必须被对齐到 16 字节边界。

可用的限定符如下:
*  `PRIVATE`， `PUBLIC`， `COMMON`和`STACK` 指定段的联合特征。`PRIVATE`段在连接时不和其他的段进行连接;`PUBLIC`和`STACK`段都会在连接时连接到一块儿;而`COMMON`段都会在同一个地址相互覆盖，而不会一接一个连接好。
*  就象上面所描述的，`ALIGN`是用来指定段基址的低位有多少位必须为零，对齐的值必须以 2 的乘方的形式给出，从 1 到 4096;实际上，真正被支持的值只有 1，2，4，16，256 和 4096，所以如果你指定了 8，它会自动向上对齐到 16，32，64 会对齐到 128 等等。注意，对齐到 4096 字节的边界是这种格式的 PharLap 扩展，可能所有的连接器都不支持。
*  `CLASS`可以用来指定段的类型;这个特性告诉连接器，具有相同 class 的段应该在输出文件中被放到相近的地址。class 的名字可以是任何字。比如`CLASS=CODE`。
*  就象`CLASS`， `OVERLAY`通过一个作为参数的字来指定，为那些有覆盖能力的连接器提供覆盖信息。
*  段可以被声明为`USE16`或`USE32`，这种选择会对目标文件产生影响，同时在段内 16 位或 32位代码分开的时候，也能保证 NASM 的缺省汇编模式
*  当编写 OS/2 目标文件的时候，你应当把 32 位的段声明为`FLAT`，它会使缺省的段基址进入一个特殊的组`FLAT`，同时，在这个组不存在的时候，定义这个组。
*  obj文件格式也允许段在声明的时候，前面有一个定义的绝对段地址，尽管没有连接器知道这个特性应该怎么使用;但如果你需要的话，NASM 还是允许你声明一个段如下面形式:`SEGMENT SCREEN ABSOLUTE=0xB800``ABSOLUTE`和`ALIGN`关键字是互斥的。NASM 的缺省段属性是`PUBLIC`， `ALIGN=1`， 没有 class，没有覆盖， 并 `USE16`.

6.2.2 `GROUP`: 定义段组。
======

obj 格式也允许段被分组，所以一个单独的段寄存器可以被用来引用一个组中的

所有段。

NASM 因此提供了`GROUP`操作符，据此，你可以这样写代码:

```nasm
       segment data 
       
               ; some data 
       
       segment bss 
       
               ; some uninitialised data 
       
       group dgroup data bss
```

这会定义一个叫做`dgroup`的组，包含有段`data`和`bss`。

就象`SEGMENT`，`GROUP`会把组名定义为一个符号，所以你可以使用`var wrt data`或者`varwrt dgroup`来引用`data`段中的变量`var`，具体用哪一个取决于哪一个段值在你的当前段寄存器中。

如果你只是想引用`var`，同时，`var`被声明在一个段中，段本身是作为一个组的一部分，然后，NASM 缺省给你的`var`的偏移值是从组的基地址开始的，而不是段基址。

所以，`SEG var`会返回组基址而不是段基址。

NASM 也允许一个段同时作为多个组的一个部分，但如果你真这样做了，会产生一个警告信息。

段内同时属于多个组的那些变量在缺省状况下会属于第一个被声明的包含它的组。

一个组也不一定要包含有段;

你还是可以使用`WRT`引用一个不在组中的变量。

比如说，OS/2 定义了一个特殊的组`FLAT`，它不包含段。


6.2.3 `UPPERCASE`: 在输出文件中使大小写敏感无效。
======


尽管 NASM 自己是大小写敏感的，有些 OMF 连接器并不大小写敏感;

所以，如果NASM 能输出大小写单一的目标文件会很有用。

`UPPERCASE`操作符让所有的写入到目标文件中的组，段，符号名全部强制为大写。

在一个源文件中，NASM还是大小写敏感的;

但目标文件可以按要求被整个写成是大写的。

`UPPERCASE`写在单独一行中，不需要任何参数。


6.2.4 `IMPORT`: 导入 DLL 符号。
======


如果你正在用 NASM 写一个 DLL 导入库，`IMPORT`操作符可以定义一个从 DLL 库中导入的符号，你使用`IMPORT`操作符的时候，你仍旧需要把符号声明为`EXTERN`.`IMPORT`操作符需要两个参数，以空格分隔，它们分别是你希望导入的符号的名称和你希望导入的符号所在的库的名称，比如:

```nasm
           import  WSAStartup wsock32.dll
```

第三个参数是可选的，它是符号在你希望从中导入的链接库中的名字，这样的话，你导入到你的代码中的符号可以和库中的符号不同名，比如:

```nasm
           import  asyncsel wsock32.dll WSAAsyncSelect
```

6.2.5 `EXPORT`: 导出 DLL 符号.
======

`EXPORT`也是一个目标格式相关的操作符，它定义一个全局符号，这个符号可以被作为一个 DLL 符号被导出，如果你用 NASM 写一个 DLL 库.你可以使用这个操作符，在使用中，你仍旧需要把符号定义为`GLOBAL`.`EXPORT`带有一个参数，它是你希望导出的在源文件中定义的符号的名字.第二个参数是可选的(跟第一个这间以空格分隔)，它给出符号的外部名字，即你希望让使用这个 DLL 的应用程序引用这个符号时所用的名字.如果这个名字跟内部名字同名，可以不使用第二个参数.还有一些附加的参数，可以用来定义导出符号的一些属性.就像第二个参数， 这些参数也是以空格分隔.如果要给出这些参数，那么外部名字也必须被指定，即使它跟内部名字相同也不能省略，可用的属性如下:
*  `resident`表示某个导出符号在系统引导后一直常驻内存.这对于一些经常使用的导出符号来说，是很有用的.
*  `nodata`表示导出符号是一个函数，这个函数不使用任何已经初始化过的数据.
*  `parm=NNN`， 这里`NNN`是一个整型数，当符号是一个在 32 位段与 16 位段之间的调用门时，它用来设置参数的尺寸大小(占用多少个 wrod).
*  还有一个属性，它仅仅是一个数字，表示符号被导出时带有一个标识数字.

比如:

```nasm
           export  myfunc 
           export  myfunc TheRealMoreFormalLookingFunctionName 
           export  myfunc myfunc 1234  ; export by ordinal 
           export  myfunc myfunc resident parm=23 nodata
```

6.2.6 `..start`: 定义程序的入口点.
======

`OMF`链接器要求被链接进来的所有目标文件中，必须有且只能有一个程序入口点，当程序被运行时，就从这个入口点开始.如果定义这个入口点的目标文件是用NASM 汇编的，你可以通过在你希望的地方声明符号`..start`来指定入口点.

6.2.7 `obj`对`EXTERN`操作符的扩展.
======

如果你以下面的方式声明了一个外部符号:

```nasm
           extern  foo
```

然后以这样的方式引用`mov ax，foo`，这样只会得到一个关于 foo 的偏移地址，而且这个偏移地址是以`foo`的首选段基址为参考的(在`foo`被定义的这个模块中指定的段).所以，为了存取`foo`的内容，你实际上需要这样做:

```nasm
               mov     ax,seg foo      ; get preferred segment base 
               mov     es,ax           ; move it into ES 
               mov     ax,[es:foo]     ; and use offset `foo' from it
```

这种方式显得稍稍有点笨拙，实际上如果你知道一个外部符号可以通过给定的段或组来进行存的话，假定组`dgroup`已经在 DS 寄存器中，你可以这样写代码:

```nasm
               mov     ax,[foo wrt dgroup]
```

但是，如果你每次要存取`foo`的时候，都要打这么多字是一件很痛苦的事情;

所以NASM 允许你声明`foo`的另一种形式:

```nasm
           extern  foo:wrt dgroup
```

这种形式让 NASM 假定`foo`的首选段基址是`dgroup`;

所以，表达式`seg foo`现在会返回`dgroup`，表达式`foo`等同于`foo wrt dgroup`.缺省的`WRT`机制可以用来让外部符号跟你程序中的任何段或组相关联.他也可以被运用到通用变量上，参阅 6.2.8.

6.2.8 `obj`对`COMMON`操作符的扩展.
======

`obj`格式允许通用变量为 near 或 far;

NASM 允许你指定你的变量属于哪一类，语法如下:

```nasm
       common  nearvar 2:near   ; `nearvar' is a near common 
       common  farvar  10:far   ; and `farvar' is far
```

通用变量可能会大于 64Kb，所以 OMF 可以把它们声明为一定数量的指定的大小的元素.比如，10byte 的 far 通用变量可以被声明为 10 个 1byte 的元素，5 个 2byte 的元素，或2 个 5byte 的元素，或 1 个 10byte 的元素.有些`OMF`链接器需要元素的 size，同时需要变量的 size，当在多个模块中声明通用变量时可以用来进行匹配.所以 NASM 必须允许你在你的 far 通用变量中指定元素的 size.这可以通过下面的语法实现:

```nasm
       common  c_5by2  10:far 5        ; two five-byte elements 
       common  c_2by5  10:far 2        ; five two-byte elements
```

如果元素的 size 没有被指定，缺省值是 1.还有，如果元素 size 被指定了，那么`far`关键字就不需要了，因为只有 far 通用变量是有元素 size 的.所以上面的声明等同于:

```nasm
       common  c_5by2  10:5            ; two five-byte elements 
       common  c_2by5  10:2            ; five two-byte elements
```

这种扩展的特性还有，`obj`中的`COMMON`操作符还可以象`EXTERN`那样支持缺省的`WRT`指定，你也可以这样声明:

```nasm
       common  foo     10:wrt dgroup 
       common  bar     16:far 2:wrt data 
       common  baz     24:wrt data:6
```
