4.4 条件汇编
======

跟 C 预处理器相似，NASM允许对一段源代码只在某特定条件满足时进行汇编，关于这个特性的语法就像下面所描述的:

```nasm
       %if<condition> 
           ; if <condition>满足时接下来的代码被汇编。 
       %elif<condition2> 
           ; 当 if<condition>不满足，而<condition2>满足时，该段代码被汇编。
       %else 
           ; 当<condition>跟<condition2>都不满足时，该段代码被汇编。
       %endif
```
`%else`跟`%elif`子句都是可选的，你也可以使用多于一个的`%elif`子句。

## 4.4.1 `%ifdef`: 测试单行宏是否存在。

`%ifdef MACRO`可以用来开始一个条件汇编块，跟在它后面的代码当且仅当一个叫做`MACRO`单行宏被定义时才会被会汇编。

如果没有定义，那么`%elif`和`%else`块会被处理。

比如，当调试一个程序时，你可能希望这样写代码:;

```nasm
                 ; perform some function 
       %ifdef DEBUG 
                 writefile 2,"Function performed successfully",13,10 
       %endif 
                 ; go and do something else
```

你可以通过使用命令行选项`-dDEBUG`来建立一个处理调试信息的程序，或不使用该选项来产生最终发布的程序。

你也可以测试一个宏是否没有被定义，这可以使用`%ifndef`。

你也可以在`%elif`块中测试宏定义，使用`%elifdef`和`%elifndef`即可。

## 4.4.2 `ifmacro`: 测试多行宏是否存在。

除了是测试多行宏的存在的，`%idmacro`操作符的工作方式跟`%ifdef`是一样的。

比如，你可能在编写一个很大的工程，而且无法控制存在链接库中的宏。

你可能需要建立一个宏，但必须先确保这个宏没有被建立过，如果被建立过了，你需要为你的宏换一个名字。

如果你定义的一个有特定参数个数与宏名的宏与现有的宏会产生冲突，那么`%ifmacro`会返回真。

比如:

```nasm
       %ifmacro MyMacro 1-3 
       
            %error "MyMacro 1-3" causes a conflict with an existing macro. 
       
       %else 
       
            %macro MyMacro 1-3 
       
                    ; insert code to define the macro 
       
            %endmacro 
       
       %endif
```

如果没有现有的宏会产生冲突，这会建立一个叫`MyMacro 1-3"的宏，如果会有冲突，那么就产生一条警告信息。

你可以通过使用`%ifnmacro`来测试是否宏不存在。

还可以使用`%elifmacro`和`%elifnmacro`在`%elif`块中测试多行宏。

## 4.4.3 `%ifctx`: 测试上下文栈。

当且仅当预处理器的上下文栈中的顶部的上下文的名字是`ctxname`时，条件汇编指令`%ifctx ctxname`会让接下来的语句被汇编。

跟`%ifdef`一样，它也有`%ifnctx`，`%elifctx`，`%elifnctx`等形式。

关于上下文栈的更多细节，参阅 4.7， 关于`%ifctx`的一个例子，参阅 4.7.5.

## 4.4.4 `%if`: 测试任意数值表达式。

当且仅当数值表达式`expr`的值为非零时，条件汇编指令`%if expr`会让接下来的语句被汇编。

使用这个特性可以确定何时中断一个`%rep`预处理器循环，例子参阅 4.5。

`%if`和`%elif`的表达式是一个临界表达式(参阅 3.8)`%if` 扩展了常规的 NASM 表达式语法，提供了一组在常规表达式中不可用的相关操作符。

操作符`=`，`<`，`>`，`<=`，`>=`和`<>`分别测试相等，小于，大于，小于等于，大于等于，不等于。

跟 C 相似的形式`==`和`!=`作为`=`，`<>`的另一种形式也被支持。

另外，低优先级的逻辑操作符`&&`，`^^`，和`||`作为逻辑与，逻辑异或，逻辑或也被支持。

这些跟 C 的逻辑操作符类似(但 C 没有提供逻辑异或)，这些逻辑操作符总是返回 0 或 1，并且把任何非零输入看作 1(所以，比如， `^^`它会在它的一个输入是零，另一个非零的时候，总返回 1)。

这些操作符返回 1 作为真值，0 作为假值。

## 4.4.5 `%ifidn` and `%ifidni`: 测试文本相同。

当且仅当`text1`和`text2`在作为单行宏展开后是完全相同的一段文本时，结构`%ifidn text1，text2`会让接下来的一段代码被汇编。

两段文本在空格个数上的不同会被忽略。

`%ifidni`和`%ifidn`相似，但是大小写不敏感。

比如，下面的宏把一个寄存器或数字压栈，并允许你把 IP 作为一个真实的寄存器使用:

```nasm
       %macro  pushparam 1 
       
         %ifidni %1,ip 
               call    %%label 
         %%label: 
         %else 
               push    %1 
         %endif 
       
       %endmacro
```

就像大多数的`%if`结构，`%ifidn`也有一个`%elifidn`，并有它的反面的形式`%ifnidn`，`%elifnidn`.相似的，`%ifidni`也有`%elifidni`，`%ifnidni`和`%elifnidni`。

## 4.4.6 `%ifid`， `%ifnum`， `%ifstr`: 测试记号的类型。

有些宏会根据传给它们的是一个数字，字符串或标识符而执行不同的动作。

比如，一个输出字符串的宏可能会希望能够处理传给它的字符串常数或一个指向已存在字符串的指针。

当且仅当在参数列表的第一个记号存在且是一个标识符时，条件汇编指令`%ifid`会让接下来的一段代码被汇编。

`%ifnum`相似。

但测试记号是否是数字;

`%ifstr`测试是否是字符串。

比如，4.3.3 中定义的宏`writefile`可以用`%ifstr`作进一步改进，如下:

```nasm
      %macro writefile 2-3+ 
       
         %ifstr %2 
               jmp     %%endstr 
           %if %0 = 3 
             %%str:    db      %2,%3 
           %else 
             %%str:    db      %2 
           %endif 
             %%endstr: mov     dx,%%str 
                       mov     cx,%%endstr-%%str 
         %else 
                       mov     dx,%2 
                       mov     cx,%3 
         %endif 
                       mov     bx,%1 
                       mov     ah,0x40 
                       int     0x21 
       
       %endmacro
```

这个宏可以处理以下面两种方式进行的调用:

```nasm
               writefile [file], strpointer, length 
               writefile [file], "hello", 13, 10
```

在第一种方式下，`strpointer`是作为一个已声明的字符串的地址，而`length`作为它的长度;

第二种方式中，一个字符串被传给了宏，所以宏就自己声明它，并为它分配地址和长度。

注意，`%ifstr`中的`%if`的使用方式:它首先检测宏是否被传递了两个参数(如果是这样，那么字符串就是一个单个的字符串常量，这样`db %2`就足够了)或者更多(这样情况下，除了前两个参数，后面的全部参数都要被合并到`%3`中，这就需要`db %2，%3`了。

)常见的`%elifXXX`，`%ifnXXX`和`%elifnXXX`/版本在`%ifid`，`%ifnum`，和`%ifstr`中都是存在的。

## 4.4.7 `%error`: 报告用户自定义错误。

预处理操作符`%error`会让 NASM 报告一个在汇编时产生的错误。

所以，如果别的用户想要汇编你的源代码，你必须保证他们用下面的代码定义了正确的宏:

```nasm
       %ifdef SOME_MACRO 
           ; do some setup 
       %elifdef SOME_OTHER_MACRO 
           ; do some different setup 
       %else 
           %error Neither SOME_MACRO nor SOME_OTHER_MACRO was defined. 
       %endif
```

然后，任何不理解你的代码的用户都会被汇编时得到关于他们的错误的警告信息，不必等到程序在运行时再出现错误却不知道错在哪儿。