8.1 与 32 位 C 代码之间的接口。
======


在 7.4 中有很多关于与 16 位 C 代码之间接口的讨论，这些东西有很多在 32 位代码中仍有用。

但已经不必担心内存模式和段的问题了，这把问题简化了很多。


## 8.1.1 外部符号名。


大多数 32 位的 C 编译器共享 16 位编译器的转化机制，即它们定义的所有的全局符号(函数与数据)的名字在 C 程序中出现时由一个下划线加上名字组成。

但是，并不是他们中的所有的都这样做::`ELF`标准指出 C 符号在汇编语言中不含有一个前导的下划线。

老的 Linux`a.out`C 编译器，所有的`Win32`编译器，`DJGPP`和`NetBSD``FreeBSD`都使用前导的下划线;

对于这些编译器来讲，7.4.1 中给出的宏`cextern`和`cglobal`还会正常工作。

对于`ELF`来讲，下划线是没有必要的。


## 8.1.2 函数定义和函数调用。

32 位程序中的 C 调用转化如下所述。

在下面的描述中，_caller_和_callee_用来 o表示调用函数和被调用函数。

*  caller 把函数的参数按相反的顺序(从右到左，这样的话，第一个参数被最后一个压栈)依次压栈
*  然后，caller 执行一个 near`CALL`指令把控制权传给 callee。
*  callee 接受控制权，然后一般会(但这实际上不是必须的，如果函数不需要存取它的参数就不用)开始先存储`ESP`的值到`EBP`中，这样就可以使用`EBP`作为一个基指针去栈中寻找参数。但是，这一点也可以放在 caller 中做，所以，调用转化中`EBP`必须被 C 函数保存起来。因为 callee 要把`EBP`设置为一个框架指针来使用，它必须把先前的值给保存起来。
*  然后，callee 就可以通过与`EBP`相关的方式来存取它的参数了。在[EBP]处的双字拥有刚刚被压栈的`EBP`的前一个值;接下来的双字，在[EBP+4]TH ，是被`CALL`指令隐式压入的返回地址。后面才是参数开始的地方，在[EBP+8]处。因为最左边的参数最后一个被压栈，在[EBP]的这个偏移地址上就可以被取得;剩下的参数依次存在后面，以连续增长的偏移值存放。这样，在一个如`printf`的带有一定数量参数的函数中，以相反的顺序把参数压栈意味着函数可以知道到哪儿去找它的第一个参数，这个参数可以告诉它总共有多少参数，它们的类型是什么。
*  callee 可能也希望能够再次减小`ESP`的值，以为本地变量开辟本地空间，这些变量然后就可以通过`EBP`的负偏移来获取。
*  callee 如果需要返回给 caller 一个值，需要根据这个值的 size 把它放在`AL`，`AX`或`EAX`中。浮点数在`ST0`中返回。
*  一旦 callee 完成了处理，如果它定义的局部栈变量，它就从`EBP`中恢复`ESP`，然后弹出前一个`EBP`的值，并通过`RET`返回。
*  当 caller 从 callee 那里取回了控制权，函数的参数还是放在栈中，所以，它通常给`ESP`加上一个立即常数以移除参数(而不是执行一系列的`pop`指令)。


这样，如果一个函数如果因为意外，使用了错误的参数个数，栈还是会返回到正常状态，因为 caller 知道多少参数被压栈了，并可以正确的移除。对于 Win32 程序使用的 Windows API 调用，有另一个可选的调用转化，对于那些被 Windows API 调用的函数(称为 windows 过程)也一样:他们遵循一个被微软叫做`__stdcall`的转化。这跟 Pascal 的转化比较接近，在这里，callee 通过给`RET`指令传递一个参数来清除栈。但是，参数还是以从右到左的顺序被压栈。

这样，你可以象下面这样定义一个 C 风格的函数:

```nasm
       global  _myfunc 
       
       _myfunc: 
               push    ebp 
               mov     ebp,esp 
               sub     esp,0x40        ; 64 bytes of local stack space 
               mov     ebx,[ebp+8]     ; first parameter to function 
       
               ; some more code 
       
               leave                   ; mov esp,ebp / pop ebp 
               ret
```

另一方面，如果你要从你的汇编代码中调用一个 C 函数，你可以象下面这样写代码:

```nasm
       extern  _printf 
       
               ; and then, further down... 
       
               push    dword [myint]   ; one of my integer variables 
               push    dword mystring  ; pointer into my data segment 
               call    _printf 
               add     esp,byte 8      ; `byte' saves space 
       
               ; then those data items... 
       
       segment _DATA 
       
       myint       dd   1234 
       mystring    db   'This number -> %d <- should be 1234',10,0
```

这段代码等同于下面的 C 代码

```nasm
           int myint = 1234; 
           printf("This number -> %d <- should be 1234\n", myint);
```

## 8.1.3 获取数据元素。

要想获取一个 C 变量的内容，或者声明一个 C 可以获取的变量，你必须把这个变量声明为`GLOBAL`或`EXTERN`(再次提醒，变量名前需要加上一个下划线，就象8.1.1 中所描述的)，这样，一个被声明为`int i`的 C 变量可以从汇编语言中这样

获取:

```nasm
                 extern _i 
                 mov eax,[_i]
```

而要定个一个 C 程序可以获取的你自己的变量`extern int j`，你可以这样做(确定你正在`_DATA`中)

```nasm
                 global _j 
       _j        dd 0
```

要获取 C 数组，你必须知道数组的元素的 size。

比如，`int`变量是 4bytes 长，所以，如果一个 C 程序声明了一个数组`int a[10]`，你可以使用代码`mov ax，[_a+12]来存取变量`a[3]`。

(字节偏移 12 是通过数组下标 3 乘上数组元素的 size4 得到的)。

基于 C 的 32 位编译器上的数据的 size 如下:1 for `char`，2 for `short`， 4 for `int`， `long` and `float`， and 8for `double`.Pointers， 32 位的地址也是 4 字节长。

要获取 C 的数据结构体，你必须知道从结构体的基地址到你所需要的域之间的偏移值。

你可以把 C 的结构体定义转化成 NASM 的结构体定义(使用`STRUC`)，或者计算得到这个偏移值，然后使用它。

以上面任何一种方式实现，你都需要阅读你的 C 编译器的手册找出它是如何组织结构体数据的。

NASM 在它的`STRUC`宏中没有给出任何特定的对齐规则，所以如果 C 编译器产生结构体，你必须自己指定对齐规则。

你可能发现类似下面的结构体:

```nasm
       struct { 
           char c; 
           int i; 
       } foo;
```

可能是 8 字节长，而不是 5 字节，因为`int`域会被对齐到 4bytes 边界。

但是，这种排布特性有时会是 C 编译器的一个配置选项，可以使用命令行选项或`#progma`行来实现，所以你必须找出你自己的编译器是如何做的。


## 8.1.4 `c32.mac`: 与 32 位 C 接口的帮助宏。

在 NASM 的包中，在`misc`子目录中，有一个宏文件`c32.mac`。

它定义了三个宏:`proc`，`arg`和`endproc`。

它们被用来定义 C 风格的过程，它们会自动产生很多代码，并跟踪调用转化过程。

使用这些宏的一个汇编函数的例子如下:

```nasm
       proc    _proc32 
       
       %$i     arg 
       %$j     arg 
               mov     eax,[ebp + %$i] 
               mov     ebx,[ebp + %$j] 
               add     eax,[ebx] 
       
       endproc
```

它把函数`_proc32`定义成一个带有两个参数的过程，第一个(`i`)是一个整型数，第二个(`j`)是一个指向整型数的指针，它返回`i+*j`。

注意，宏`arg`展开后的第一行有个`EQU`，因为在宏调用行的前面的那个 label被加到了第一行上，`EQU`行就可以正常工作了，它把`%Si`定义为一个以`BP`为基址的偏移值。

一个 context-local 变量在这里被使用，被`proc`宏压栈，然后被`endproc`宏出栈，所以，同样的参数名在后来的过程中还是可以使用，当然你不一定要那样做。

`arg`带有一个可选的参数，给出参数的 size。

如果没有 size 给出，缺省的是 4，因为很多函数参数都会是`int`类型或者是一个指针。
