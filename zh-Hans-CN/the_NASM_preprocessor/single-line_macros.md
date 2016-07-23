4.1 单行的宏。
======

## 4.1.1 最常用的方式: `%define`

单行的宏是以预处理指令`%define`定义的。

定义工作同 C 很相似，所以你可以这样做:

```nasm
      %define ctrl    0x1F & 
       %define param(a,b) ((a)+(a)*(b)) 
       
               mov     byte [param(2,ebx)], ctrl 'D'
```

会被扩展为:

```nasm
               mov     byte [(2)+(2)*(ebx)], 0x1F & 'D'
```

当单行的宏被扩展开后还含有其它的宏时，展开工作会在执行时进行，而不是定义时，如下面的代码:

```nasm
       %define a(x)    1+b(x) 
       %define b(x)    2*x 
       
               mov     ax,a(8)
```

会如预期的那样被展开成`mov ax， 1+2*8`， 尽管宏`b`并不是在定义宏 a的时候定义的。

用`%define`定义的宏是大小写敏感的:在代码`%define foo bar`之后，只有`foo`会被扩展成`bar`:`Foo`或者`FOO`都不会。

用`%idefine`来代替`%define`(i 代表`insensitive`)，你可以一次定义所有的大小写不同的宏。

所以`%idefine foo bar`会导致`foo`，`FOO`，`Foo`等都会被扩展成`bar`。

当一个嵌套定义(一个宏定义中含有它本身)的宏被展开时，有一个机制可以检测到，并保证不会进入一个无限循环。

如果有嵌套定义的宏，预处理器只会展开第一层，因此，如果你这样写:

```nasm
       %define a(x)    1+a(x) 
       
               mov     ax,a(3)
```

宏 `a(3)`会被扩展成`1+a(3)`，不会再被进一步扩展。

这种行为是很有用的，有关这样的例子请参阅 8.1。

你甚至可以重载单行宏:如果你这样写:

```nasm
       %define foo(x)   1+x 
       %define foo(x,y) 1+x*y
```

预处理器能够处理这两种宏调用，它是通过你传递的参数的个数来进行区分的，所以`foo(3)`会变成`1+3`，而`foo(ebx，2)`会变成`1+ebx*2`。

尽管如此，但如果你定义了:

```nasm
       %define foo bar
```

那么其他的对`foo`的定义都不会被接受了:一个不带参数的宏定义不允许对它进行带有参数进行重定义。

但这并不能阻止单行宏被重定义:你可以像这样定义，并且工作得很好:

```nasm
       %define foo bar
```

然后在源代码文件的稍后位置重定义它:

```nasm
       %define foo baz
```

然后，在引用宏`foo`的所有地方，它都会被扩展成最新定义的值。

这在用`%assign`定义宏时非常有用(参阅 4.1.5)你可以在命令行中使用`-d`选项来预定义宏。参阅 2.1.11

## 4.1.2 %define 的增强版: `%xdefine`

与在调用宏时展开宏不同，如果想要调用一个嵌入有其他宏的宏时，使用它在被定义的值，你需要`%define`不能提供的另外一种机制。

解决的方案是使用`%xdefine`，或者它的大小写不敏感的形式`%xidefine`。

假设你有下列的代码:

```nasm
       %define  isTrue  1 
       %define  isFalse isTrue 
       %define  isTrue  0 
       
       val1:    db      isFalse 
       
       %define  isTrue  1 
       
       val2:    db      isFalse
```

在这种情况下，`val1`等于 0，而`val2`等于 1。

这是因为，当一个单行宏用`%define`定义时，它只在被调用时进行展开。

而`isFalse`是被展开成`isTrue`，所以展开的是当前的`isTrue`的值。

第一次宏被调用时，`isTrue`是 0，而第二次是 1。

如果你希望`isFalse`被展开成在`isFalse`被定义时嵌入的`isTrue`的值，你必须改写上面的代码，使用`%xdefine`:

```nasm
       %xdefine isTrue  1 
       %xdefine isFalse isTrue 
       %xdefine isTrue  0 
       
       val1:    db      isFalse 
       
       %xdefine isTrue  1 
       
       val2:    db      isFalse
```

现在每次`isFalse`被调用，它都会被展开成 1，而这正是嵌入的宏`isTrue`在`isFalse`被定义时的值。

## 4.1.3 : 连接单行宏的符号: `%+`

一个单行宏中的单独的记号可以被连接起来，组成一个更长的记号以待稍后处理。

这在很多处理相似的事情的相似的宏中非常有用。

举个例子，考虑下面的代码:

```nasm
       %define BDASTART 400h                ; Start of BIOS data area

       struc   tBIOSDA                      ; its structure 
               .COM1addr       RESW    1 
               .COM2addr       RESW    1 
               ; ..and so on 
       endstruc
```

现在，我们需要存取 tBIOSDA 中的元素，我们可以这样:

```nasm
               mov     ax,BDASTART + tBIOSDA.COM1addr 
               mov     bx,BDASTART + tBIOSDA.COM2addr
```

如果在很多地方都要用到，这会变得非常的繁琐无趣，但使用下面的宏会大大减小打字的量:;

 Macro to access BIOS variables by their names (from tBDA):

```nasm
               mov     ax,BDA(COM1addr) 
               mov     bx,BDA(COM2addr)
```

 使用这个特性，我们可以简单地引用大量的宏。(另外，还可以减少打字错误)。

## 4.1.4 取消宏定义: `%undef`

单行的宏可以使用`%undef`命令来取消。

比如，下面的代码:

```nasm
       %define foo bar 
       %undef  foo 
       
               mov     eax, foo
```

会被展开成指令`mov eax， foo`，因为在`%undef`之后，宏`foo`处于无定义状态。

那些被预定义的宏可以通过在命令行上使用`-u`选项来取消定义，参阅 2.1.13

## 4.1.5 预处理器变量 : `%assign`

定义单行宏的另一个方式是使用命令`%assign`(它的大小写不敏感形式是%iassign，它们之间的区别与`%idefine`，`%idefine`之间的区别完全相同)。

`%assign`被用来定义单行宏，它不带有参数，并有一个数值型的值。

它的值可以以表达式的形式指定，并要在`%assing`指令被处理时可以被一次计算出来，就像`%define`，`%assign`定义的宏可以在后来被重定义，所以你可以这样做:

```nasm
       %assign i i+1
```

以此来增加宏的数值`%assing`在控制`%rep`的预处理器循环的结束条件时非常有用:请参阅 4.5 的例子。

另外的关于`%assign`的使用在 7.4 和 8.1 中的提到。

赋给`%assign`的表达式也是临界表达式(参阅 3.8)，而且必须可被计算成一个纯数值型(不能是一个可重定位的指向代码或数据的地址，或是包含在寄存器中的一个值。)