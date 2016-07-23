4.9 TASM 兼容预处理指令。
======

接下来的预处理操作符只有在用`-t`命令行开关把 TASM 兼容模式打开的情况下才可以使用(这个开关在 2.1.16 介绍过)

*  `%arg` (见 4.9.1 节)
*  `%stacksize` (见 4.9.2 节)
*  `%local` (见 4.9.3 节)

## 4.9.1 `%arg`操作符`%arg`操作符用来简化栈上的参数传递操作处理。

基于栈的参数传递在很多高

级语言中被使用，包括 C，C++和 Pascal。

而 NASM 企图通过宏来实现这种功能(参阅 7.4.5)，它的语法使用上不是很舒服，而且跟 TASM 之间是不兼容的。

这里有一个例子，展示了只通过宏`%arg`来处理:

```nasm
       some_function: 
       
           %push     mycontext        ; save the current context 
           %stacksize large           ; tell NASM to use bp 
           %arg      i:word, j_ptr:word 
       
               mov     ax,[i] 
               mov     bx,[j_ptr] 
               add     ax,[bx] 
               ret 
       
           %pop                       ; restore original context
```

这跟在 7.4.5 中定义的过程很相似，把 j_ptr 指向的值加到 i 中，然后把相加的结果在 AX 中返回，对于`push`和`pop`的展开请参阅 4.7.1 关于上下文栈的使用。

## 4.9.2 `%stacksize`指令。

`%stacksize`指令是跟`%arg`和`%local`指令结合起来使用的。

它告诉 NASM为`%arg`和`%local`使用的缺省大小。

`%stacksize`指令带有一个参数，它是`flat`，`large`或`small`。

```nasm
       %stacksize flat
```

这种形式将使 NASM 使用相对于`ebp`的基于栈的参数地址。

它假设使用一个近调用来得到这个参数表。

(比如，eip 被压栈)

```nasm
       %stacksize large
```

而这种形式会用`bp`来进行基于栈的参数寻址，假设使用了一个远调用来获得这个地址(比如，ip 和 cs 都会被压栈)。

```nasm
       %stacksize small
```

这种形式也使用`bp`来进行基于栈的参数寻址，但它跟`large`不同，因为他假设 bp 的旧值已经被压栈。

换句话说，你假设 bp，ip 和 cs 正在栈顶，在它们下面的所有本地空间已经被`ENTER`指令开辟好了。

当和`%local`指令结合的时候，这种形式特别有用。

## 4.9.3 `%local`指令。

`%local`指令用来简化在栈框架中进行本地临时栈变量的分配。

C 语言中的自动本地变量是这种类型变量的一个例子。

`%local`指令跟`%stacksize`一起使用的时候特别有用。

并和`%arg`指令保持兼容。

它也允许简化对于那些用`ENTER`指令分配在栈中的变量的引用(关于 ENTER 指令，请参况 B.4.65)。

这里有一个关于它们的使用的例子:

```nasm
       silly_swap: 
       
           %push mycontext             ; save the current context 
           %stacksize small            ; tell NASM to use bp 
           %assign %$localsize 0       ; see text for explanation 
           %local old_ax:word, old_dx:word 
       
               enter   %$localsize,0   ; see text for explanation 
               mov     [old_ax],ax     ; swap ax & bx 
               mov     [old_dx],dx     ; and swap dx & cx 
               mov     ax,bx 
               mov     dx,cx 
               mov     bx,[old_ax] 
               mov     cx,[old_dx] 
               leave                   ; restore old bp 
               ret                     ; 
       
           %pop                        ; restore original context
```

变量`%$localsize`是在`%local`的内部使用，而且必须在`%local`指令使用前，被定义在当前的上下文中。

不这样做，在每一个`%local`变量声明的地方会引发一个表达式语法错误。

它然后可以用在一条适当的 ENTER 指令中。