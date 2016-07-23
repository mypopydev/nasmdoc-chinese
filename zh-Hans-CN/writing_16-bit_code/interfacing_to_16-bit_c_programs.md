7.4 与 16 位 C 程序之间的接口。
======


本章介绍编写调用 C 程序的汇编过程或被 C 程序调用的汇编过程的基本方法。

要做到这一点，你必须把汇编模块写成`.OBJ`文件，然后把它和你的 C 模块一起连接，产生一个混合语言程序。


## 7.4.1 外部符号名。

C 编译器对所有的全局符号(函数或数据)的名字有一个转化，它们被定义为在名字前面加上一个下划线，就象在 C 程序中出现的那样。

所以，比如，一个 C 程序的函数`printf`对汇编语言程序中来说，应该是`_printf`。

你意味着在你的汇编程序中，你可以定义前面不带下划线的符号，而不必担心跟 C 中的符号名产生冲突。

如果你觉得下划线不方便，你可以定义一个宏来替换`GLOBAL`和`EXTERN`操作符:

```nasm
       %macro  cglobal 1 
       
         global  _%1 
         %define %1 _%1 
       
       %endmacro 
       
       %macro  cextern 1 
       
         extern  _%1 
         %define %1 _%1 
       
       %endmacro
```

(这些形式的宏一次只带有一个参数;

`%rep`结构可以解决这个问题)。

如果你象下面这样定义一个外部符号:

```nasm
       cextern printf
```

这个宏就会被展开成:

```nasm
       extern  _printf 
       %define printf _printf
```

然后，你可用把`printf`作为一个符号来引用，预处理器会在必要的时候在前面加上一个下划线。

`cglobal`宏以相似的方式工作。

## 7.4.2 内存模式。

NASM 没有提供支持各种 C 的内存模式的直接机制;

你必须自己记住你在何种模式下工作。

这意味着你自己必须跟踪以下事情:

*  在使用单个代码段的模式中(tiny small 和 compact)函数都是 near 的，这表示函数指针在作为一个函数参数存入数据段或压栈时，有 16 位长并只包含一个偏移域(CS 寄存器中的值从来不改变，总是给出函数地址的段地真正部分)，函数调用就使用普通的 near`CALL`指令，返回使用`RETN`(在 NASM 中，它跟`RET`同义)。这意味着你在编写你自己的过程时，应当使用`RETN`返回，你调用外部 C 过程时，可以使用 near 的`CALL`指令。
*  在使用多于一个代码段的模块中(medium， large 和 huge)函数是 far 的，这表示函数指针是 32 位长的(包含 16 位的偏移值和紧跟着的 16 位段地址)，这种函数使用`CALL FAR`进行调用(或者`CALL seg:offset`)而返回使用`RETF`。同样的，你编写自己的过程时，应当使用`RETF`，调用外部 C 过程应当使用`CALL FAR`。
*  在使用单个数据段的模块中(tiny， small 和 medium)，数据指针是 16 位长的，只包含一个偏移域(`DS`寄存器的值不改变，总是给出数据元素的地址的段地址部分)。
*  在使用多于一个数据段的模块中(compact， large 和 huge)，数据指针是 32 位长的，包含一个 16 位的偏移跟上一佧 16 位的段地址。你还是应当小心，不要随便改变了 ds 的值而没有恢复它，但是 ES 可以被随便用来存取 32 位数据指针的内容。

## 7.4.3 函数定义和函数调用。

16 位程序中的 C 调用转化如下所示。

在下面的描述中，_caller_和_callee_分别表示调用者和被调用者。

*  caller 把函数的参数按相反的顺序压栈，(从右到左，所以第一个参数被最后一个压栈)。
*  caller 然后执行一个`CALL`指令把控制权交给 callee。根据所使用的内存模式，`CALL`可以是 near 或 far。
*  callee 接收控制权，然后一般会(尽管在没有带参数的函数中，这不是必须的)在开始的时候把`SP`的值赋给`BP`，然后就可以把`BP`作为一个基准指针用以寻找栈中的参数。当然，这个事情也有可能由caller 来做，所以，关于`BP`的部分调用转化工作必须由 C 函数来完成。因此 callee 如果要把`BP`设为框架指针，它必须把先前的 BP 值压栈。
*  然后 callee 可能会以`BP`相关的方式去存取它的参数。在[BP]中存有 BP在压栈前的那个值;下一字 word，在[BP+2]处，是返回地址的偏移域，由`CALL`指令隐式压入。在一个 small 模式的函数中。在[BP+4]处是参数开始的地方;在 large 模式的函数中，返回地址的段基址部分存在[BP+4]的地方，而参数是从[BP+6]处开始的。最左边的参数是被后一个被压入栈的，所以在`BP`的这点偏移值上就可以被取到;其他参数紧随其后，偏移地址是连续的.这样，在一个象`printf`这样的带有一定数量的参数的函数中，以相反的顺序把参数压栈意味着函数可以知道从哪儿获得它的第一个参数，这个参数可以告诉接接下来还有多少参数，和它们的类型分别是什么.
*  callee 可能希望减小`sp`的值，以便在栈中分配本地变量，这些变量可以用`BP`负偏移来进行存取.
*  callee 如果想要返回给 caller 一个值，应该根据这个值的大小放在`AL`，`AX`或`DX:AX`中.如果是浮点类型返回值，有时(看编译器而定)会放在`ST0`中.
*  一旦 callee 结束了处理，它如果分配过了本地空间，就从`BP`中恢复`SP`的值，然后把原来的`BP`值出栈，然后依据使用的内存模式使用`RETN`或`RETF`返回值.
*  如果 caller 从 callee 中又重新取回了控制权，函数的参数仍旧在栈中，所以它需要加一个立即常数到`SP`中去，以移除这些参数(不用执行一系列的 pop 指令来达到这个目的).这样，如果一个函数因为匹配的问题偶尔被以错误的参数个数来调用，栈还是会返回一个正常的状态，因为 caller 知道有多少个参数被压了，它会把它们正确的移除.这种调用转化跟 Pascal 程序的调用转化是没有办法比较的(在 7.5.1 描述).pascal拥有一个更简单的转化机制，因为没有函数拥有可变数目的参数.所以 callee 知道传递了多少参数，它也就有能力自己来通过传递一个立即数给`RET`或`RETF`指令来移除栈中的参数，所以 caller 就不必做这个事情了.同样，参数也是以从左到右的顺序被压栈的，而不是从右到左，这意味着一个编译器可以更方便地处理。

这样，如果你想要以 C 风格定义一个函数，应该以下面的方式进行:这个例子是在 small 模式下的。


```nasm
       global  _myfunc 
       
       _myfunc: 
               push    bp 
               mov     bp,sp 
               sub     sp,0x40         ; 64 bytes of local stack space 
               mov     bx,[bp+4]       ; first parameter to function 
       
               ; some more code 
       
               mov     sp,bp           ; undo "sub sp,0x40" above 
               pop     bp 
               ret
```

在巨模式下，你应该把`RET`替换成`RETF`，然后应该在[BP+6]的位置处寻找第一个参数，而不是[BP+4].当然，如果某一个参数是一个指针的话，那参数序列的偏移值会因为内存模式的改变而改变:far 指针作为一个参数时在栈中占用4bytes，而 near 指针只占用两个字节。

另一方面，如果从你的汇编代码中调用一个 C 函数，你应该做下面的一些事情:

```nasm
       extern  _printf 
       
             ; and then, further down... 
       
             push    word [myint]        ; one of my integer variables 
             push    word mystring       ; pointer into my data segment 
             call    _printf 
             add     sp,byte 4           ; `byte' saves space 
       
             ; then those data items... 
       
       segment _DATA 
       
       myint         dw    1234 
       mystring      db    'This number -> %d <- should be 1234',10,0
```

这段代码在 small 内存模式下等同于下面的 C 代码:

```nasm
           int myint = 1234; 
           printf("This number -> %d <- should be 1234\n", myint);
```

在 large 模式下，函数调用代码可能更象下面这样。

在这个例子中，假设 DS 已经含有段`_DATA`的段基址，你首先必须初始化它:

```nasm
             push    word [myint] 
             push    word seg mystring   ; Now push the segment, and... 
             push    word mystring       ; ... offset of "mystring" 
             call    far _printf 
             add    sp,byte 6
```

这个整型值在栈中还是占用一个字的空间，因为 large 模式并不会影响到`int`数据类型的 size.printf 的第一个参数(最后一个压栈)，是一个数据指针，所以含有一个段基址和一个偏移域。

在内存中，段基址应该放在偏移域后面，所以，必须首先被压栈。

(当然，`PUSH DS`是一个取代`PUSH WORD SEG mystring`的更短的形式，如果 DS 已经被正确设置的话)。

然后，实际的调用变成了一个 far 调用，因为在 large 模式下，函数都是被 far 调用的;

调用后，`SP`必须被加上 6，而不是4，以释放压入栈中的参数。


7.4.4 存取数据元素。
======


要想获得一个 C 变量的内容，或者声明一个 C 语言可以存取的变量，你只需要把变量名声明为`GLOBAL`或`EXTERN`即可。

(再次提醒，就象在 7.4.1 中所介绍的，变量名前需要加上一个下划线)这样，一个在 C 中声明的变量`ini i`可以在汇编语中以下述方式存取:

```nasm
       extern _i 
       
               mov ax,[_i]
```

而要声明一个你自己的可以被 C 程序存取的整型变量如:`extern int j`，你可以这样做(确定你下在`_DATA`段中):

```nasm
       global  _j 
       
       _j      dw      0
```

要存取 C 的数组，你需要知道数组元素的 size.比如，`int`变量是 2byte 长，所以如果一个 C 程序声明了一个数组`int a[10]`，你可象这样存取`a[3]`:`mov ax，[_a+6]`.(字节偏移 6 是通过数组下标 3 乘上数组元素的 size2 得到的。

) 基于 C 的16 位编译器的数据 size 如下:1 for `char`， 2 for `short` and `int`， 4for `long` and `float`， and 8 for `double`.为了存取 C 的数据结构，你必须知道从结构的基地址到你所感兴趣的域的偏移地址。

你可以通过把 C 结构定义转化为 NASM 的结构定义(使用`STRUC`)，或者计算这个偏移地址然后进行相应操作。

以上述任何一种方法实现，你必须得阅读你的 C 编译器的手册去找出他是如何组织数据结构的。

NASM 在它的宏`STRUC`中不给出任何对结构体成员的对齐操作，所以你可能会发现结构体类似下面的样子:

```nasm
       struct { 
           char c; 
           int i; 
       } foo;
```


可能就是 4 字节长，而不是三个字，因为`int`域会被对齐到 2byte 边界。

但是，这种排布的特性在 C 编译器中很可能只是一个配置选项，使用命令行选项或者`#pragma`行。

所以你必须找出你的编译器是如何实现这个的。


7.4.5 `c16.mac`: 与 16 位 C 接口的帮助宏。
======


在 NASM 包中，在`misc`子目录下，是一个宏文件`c16.mac`。

它定义了三个宏`proc`，`arg`和`endproc`。

这些被用在 C 风格的过程定义中，它们自动完成了很多工作，包括对调用转化的跟踪。

(另外一种选择是，TASM 兼容模式的`arg`现在也被编译进了 NASM 的预处理器，详见 4.9)关于在汇编函数中使用这个宏的一个例子如下:

```nasm
       proc    _nearproc 
       
       %$i     arg 
       %$j     arg 
               mov     ax,[bp + %$i] 
               mov     bx,[bp + %$j] 
               add     ax,[bx] 
       
       endproc
```

这把`_nearproc`定义为一个带有两个参数的一个过程，第一个(`i`)是一个整型数，第二个(`j`)是一个指向整型数的指针，它返回`i+*j`。

注意，`arg`宏展开的第一行有一个`EQU`，而且因为在宏调用的前面的那个label 在宏展开后被加在了第一行的前面，所以`EQU`能否工作取决于`%$i`是否是一个关于`BP`的偏移值。

同时一个对于上下文来说是本地的 context-local 变量被使用，它被`proc`宏压栈，然后被`endproc`宏出栈，所以，在后来的过程中，同样的参数名还是可以使用，当然，你不一定要这么做。

宏在缺省状况下把过程代码设置为 near 函数(tiny，small 和 compact 模式代码)，你可以通过代码`%define FARCODE`产生 far 函数(medium， large 和 huge 模式代码)。

这会改变`endproc`产生的返回指令的类型，还会改变参数起始位置的偏移值。

这个宏在设置内容时，本质上并依赖数据指针是 near 或 far。

`arg`可以带有一个可选参数，给出参数的 size。

如果没有 size 给出，缺省设置为2，因为绝大多数函数参数会是`int`类型。

上面函数的 large 模式看上去应该是这个样子:

```nasm
       %define FARCODE 
       
       proc    _farproc 
       
       %$i     arg 
       %$j     arg     4 
               mov     ax,[bp + %$i] 
               mov     bx,[bp + %$j] 
               mov     es,[bp + %$j + 2] 
               add     ax,[bx] 
       
       endproc
```

这利用了`arg`宏的参数定义参数的 size 为 4，因为`j`现在是一个 far 指针。

当我们从`j`中载入数据时，我们必须同时载入一个段基址和一个偏移值。
