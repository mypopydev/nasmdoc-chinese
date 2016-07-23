4.7 上下文栈。
======

那些对一个宏定义来讲是本地的 Labels 有时候还不够强大:有时候，你需要能够在多个宏调用之间共享 label。

比如一个`REPEAT`...`UNTIL`循环，`REPEAT`宏的展开可能需要能够去引用`UNTIL`中定义的宏。

而且在使用这样的宏时，你可能还会嵌套多层循环。

NASM 通过上下文栈提供这个层次上的功能。

预处理器维护了一个包含上下文的栈，每一个上下文都有一个名字作为标识。

你可以通过指令`%push`往上下文栈中加一个新的上下文，或通过`%pop`去掉一个。

你可以定义一些只针对特定上下文来说是本地的 labels。

## 4.7.1 `%push` and `%pop`: 创建和删除上下文。

`%push`操作符用来创建一个新的上下文，然后把它放在上下文栈的顶端。

`%push`需要一个参数，它是这个上下文的名字，例如:

```nasm
       %push    foobar
```

这会把一个新的叫做`foobar`的上下文放到栈顶。

你可以在一个栈中拥有多个具有相同名字的上下文:它们之间仍旧是可以区分的。

操作符`%pop`不需要参数，删除栈顶的上下文，并把它销毁，同时也删除跟它相关的 labels。

## 4.7.2 Context-Local Labels

就像`%%foo`会定义一个对于它所在的那个宏来讲是本地的 label 一样，`%$foo`会定义一个对于当前栈顶的上下文来讲是本地的 lable。

所以，上文提到的`REPEAT`，`UNTIL`的例子可以以下面的方式实现:

```nasm
       %macro repeat 0 
       
           %push   repeat 
           %$begin: 
       
       %endmacro 
       
       %macro until 1 
       
               j%-1    %$begin 
           %pop 
       
       %endmacro
```

然后象下面这样使用它:

```nasm

               mov     cx,string 
               repeat 
               add     cx,3 
               scasb 
               until   e
```

它会扫描每个字符串中的第四个字节，以查找在 `AL` 中的字节。

如果你需要定义，或存取对于不在栈顶的上下文本地的 label，你可以使用`%$$foo`，或`%$$$foo`来存取栈下面的上下文。

## 4.7.3 Context-Local 单行宏。

NASM 也允许你定义对于一个特定的上下文是本地的单行宏，使用的方式大致相面:

```nasm
       %define %$localmac 3
```

这会定义一个对于栈顶的上下文本地的单行宏`%$localmax`，当然，在又一个`%push`操作之后，它还是可以通过`%$$localmac`来存取。

## 4.7.4 `%repl`: 对一个上下文改名。

如果你需要改变一个栈顶上下文的名字(比如，为了响应`%ifctx`)，你可以在`%pop`之后紧接着一个`%push`;

但它会产生负面效应，会破坏所有的跟栈顶上下文相关的 context-local labels 和宏。

NASM 提供了一个操作符`%repl`，它可以在不影响相关的宏与 labels 的情况下，为一个上下文换一个名字，所以你可以把下面的破坏性代码替换成另一种形式:

```nasm
       %pop 
       %push   newname
```

换成不具破坏性的版本: `%repl newname`.

## 4.7.5 使用上下文栈的例子: Block IFs

这个例子几乎使用了所有的上下文栈的特性，包括条件汇编结构`%ifctx`，它把一个块 IF 语句作为一套宏来执行:

```nasm
       %macro if 1 
       
           %push if 
           j%-1  %$ifnot 
       
       %endmacro 
       
       %macro else 0 
       
         %ifctx if 
               %repl   else 
               jmp     %$ifend 
               %$ifnot: 
         %else 
               %error  "expected `if' before `else'" 
         %endif 
       
       %endmacro 
       
       %macro endif 0 
       
         %ifctx if 
               %$ifnot: 
               %pop 
         %elifctx      else 
               %$ifend: 
               %pop 
         %else 
               %error  "expected `if' or `else' before `endif'" 
         %endif 
       
       %endmacro
```

这段代码看上去比上面的`REPEAT`和`UNTIL`宏要饱满多了。

因为它使用了条件汇编去验证宏以正确的顺序被执行(比如，不能在`if`之间调用`endif`)如果出现错误，执行`%error`。

另外，`endif`宏要处理两种不同的情况，即它可能直接跟在`if`后面，也可能跟在`else`后面。

它也是通过条件汇编，判断上下文栈的栈顶是`if`还是`else`，并据此来执行不同的动作。

`else`宏必须把上下文保存到栈中，好让`if`宏跟`endif`宏中定义的`%$ifnot`引用。

但必须改变上下文的名字，这样`endif`就可以知道这中间还有一个`else`。

这是通过`%repl`来做这件事情的。

下面是一个使用这些宏的例子:

```nasm
        cmp     ax,bx 

        if ae 
               cmp     bx,cx 

               if ae 
                       mov     ax,cx 
               else 
                       mov     ax,bx 
               endif 

        else 
               cmp     ax,cx 

               if ae 
                       mov     ax,cx 
               endif 

        endif
```

通过把在内层`if`中描述的另一个上下文压栈，放在外层`if`中的上下文的上面，这样，`else`和`endif`总能引用到匹配的`if`或`else`。

这个块-`IF`宏处理嵌套的能力相当好，