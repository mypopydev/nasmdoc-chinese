4.5 预处理器循环: `%rep`
======

虽然 NASM 的`TIMES`前缀非常有用，但是不能用来作用于一个多行宏，因为它是在 NASM 已经展开了宏之后才被处理的。

所以，NASM 提供了另外一种形式的循环，这回是在预处理器级别的:`%rep`。

操作符`%rep`和`%endrep`(`%rep`带有一个数值参数，可以是一个表达式;

`%endrep`不带任何参数)可以用来包围一段代码，然后这段代码可以被复制多次，次数由预处理器指定。

```nasm
       %assign i 0 
       %rep    64 
               inc     word [table+2*i] 
       %assign i i+1 
       %endrep
```

这段代码会产生连续的 64 个`INC`指令，从内存地址`[table]`一直增长到`[table+126]`。

对于一个复杂的终止条件，或者想要从循环中 break 出来，你可以使用`%exitrep`操作符来终止循环，就像下面这样:

```nasm
       fibonacci: 
       %assign i 0 
       %assign j 1 
       %rep 100 
       %if j > 65535 
           %exitrep 
       %endif 
               dw j 
       %assign k j+i 
       %assign i j 
       %assign j k 
       %endrep 
       
       fib_number equ ($-fibonacci)/2
```

上面的代码产生所有 16 位的 Fibonacci 数。

但要注意，循环的最大次数还是要作为一个参数传给`%rep`。

这可以防止 NASM 预处理器进入一个无限循环。

在多任务或多用户系统中，无限循环会导致内存被耗光或其他程序崩溃。