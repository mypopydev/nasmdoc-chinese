8.2 编写 NetBSD/FreeBSD/OpenBSD 和 Linux/ELF 共享库
======

`ELF`在 Linux 下取代了老的`a.out`目标文件格式，因为它包含对于地址无关代码(PIC)的支持，这可以让编写共享库变得很容易。

NASM 支持`ELF`的地址无关代码特性，所以你可以用 NASM 来编写 Linux 的`ELF`共享库。

NetBSD，和它的近亲 FreeBSD，OpenBSD，采用了一种不同的方法，它们把 PIC 支持做进了`a.out`格式。

NASM 支持这些格式，把它们叫做`aoutb`输出格式，所以你可以在 NASM 下写 BSD 的共享库。

操作系统是通过把一个库文件内存映射到一个运行进程的地址空间上的某一个点来实现载入 PIC 共享库的。

所以，库的代码段内容必须不依赖于它被载入到了内存的什么地方。

因此，你不能通过下面的代码得到你的变量:

```nasm
               mov     eax,[myvar]             ; WRONG
```

而是通过连接器提供一片内存空间，这片空间叫做全局偏移表(GOT);

GOT 被放到离你库代码的一个常量距离值的地方，所以如果你发现了你的库被载入到了什么地方(这可以通过使用`CALL`和`POP`指令而得到)，你可以得到 GOT 中的地址，然后你就可以通过这个连接器产生的在 GOT 中的入口来载入你的变量的地址。

而 PIC 共享库的数据段就没有这些限制了:因为数据段是可定局的，它必须被拷贝到内存中，而不是仅仅从库文件中作一个映射，所以一旦它被拷贝进来，它就可以被重定位。

所以你可以把一些常规的在数据段中重定位的类型用进来，而不必担心会有什么错误发生。


## 8.2.1 取得 GOT 中的地址。


每个在你的共享库中的代码模块都应当把 GOT 定义为一个导出符号:

```nasm
       extern  _GLOBAL_OFFSET_TABLE_   ; in ELF 
       extern  __GLOBAL_OFFSET_TABLE_  ; in BSD a.out
```

在你的共享库中，那些需要获取你的 data 或 BSS 段中数据的函数，你必须在它们的开头先计算 GOT 的地址。

这一般以如下形式编写这个函数:

```nasm
       func:   push    ebp 
               mov     ebp,esp 
               push    ebx 
               call    .get_GOT 
       .get_GOT: 
               pop     ebx 
               add     ebx,_GLOBAL_OFFSET_TABLE_+$$-.get_GOT wrt ..gotpc 
       
               ; the function body comes here 
       
               mov     ebx,[ebp-4] 
               mov     esp,ebp 
               pop     ebp 
               ret
```

(对于 BSD， 符号`_GLOBAL_OFFSET_TABLE`开头需要两个下划线。)这个函数的头两行只是简单的标准的 C 风格的开头，用于设置栈框架，最后的三行是标准的 C 风格的结尾，第三行，和倒数第四行，分别保存和恢复`EBS`寄存器，因为 PIC 共享库使用这个寄存器保存 GOT 的地址。

最关键的是`CALL`指令和接下来的两行代码。

`CALL`和`POP`一起用来获得.get_GOT 的地址，不用进一步知道程序被载入到什么地方(因为 call 指令是解码成跟当前的位置相关)。

`ADD`指令使用了一个特殊的 PIC 重定位类型:GOTPC重定位。

通过使用限定符`WRT ..gotpc`，被引用的符号(这里是`_GLOBAL_OFFSET_TABLE_`，一个被赋给 GOT 的特殊符号)被以从段起始地址开始的偏移的形式给出。

(实际上，`ELF`把它编码为从`ADD`的操作数域开始的一个偏移，但 NASM 把它简化了，所以你在`ELF`和`BSD`中可以用同样的方式处理。

)所以，这条指令然后加上段起始地址，然后得到 GOT 的真正的地址。

然后减去`.get_GOT`的值，当这条指令执行结束的时候，`EBX`中含有`GOT`的值。

如果你不理解上面的内容，也不用担心:因为没有必要以第二种方式来获得GOT 的地址，所以，你可以把这三条指令写成一个宏，然后就可以安全地忽略它们:

```nasm
       %macro  get_GOT 0 
       
               call    %%getgot 
         %%getgot: 
               pop     ebx 
               add     ebx,_GLOBAL_OFFSET_TABLE_+$$-%%getgot wrt ..gotpc 
       
       %endmacro
```

## 8.2.2 寻址你的本地数据元素。

得到 GOT 后，你可以使用它来得到你的数据元素的地址。

大多数变量会在你声明过的段中;

它们可以通过使用`..gotoff`来得到。

它工作的方式如下:

```nasm
               lea     eax,[ebx+myvar wrt ..gotoff]
```

表达式`myvar wrt ..gotoff`在共享库被连接进来的时候被计算，得到从 GOT地始地址开始的变量`myvar`的偏移值。

所以，把它加到上面的`EBX`中，并把它放到`EAX`中.如果你把一些变量声明为`GLOBAL`，而没有指定它们的 size 的话，它们在库中的代码模块间会被共享，但不会被从库中导出到载入它们的程序中.但们还会存在于你的常规 data 和 BSS 段中，所以通过上面的`..gotoff`机制，你可以把它们作为局部变量那样存取注意，因为 BSD 的`a.out`格式处理这种重定位类型的一种方式，在你要存取的地址处的同一个段内必须至少有一个非本地的符号.

## 8.2.3 寻址外部和通用数据元素.

如果你的库需要得到一个外部变量(对库来说是外部的，并不是对它所在的一个模块)，你必须使用`..got`类型得到它.`..got`类型，并不给你从 GOT 基地址到变量的偏移，给你的是从 GOT 基地址到一个含有这个变量地址的 GOT 入口的偏移，连接器会在构建库时设置这个 GOT 入口，动态连接器会在载入时在这个入口放上正确的地址.所以，要得到一个外部变量`extvar`的地址，并放到 EAX 中，你可以这样写:

```nasm
               mov     eax,[ebx+extvar wrt ..got]
```

这会在 GOT 的一个入口上载入`extvar`的地址.连接器在构建共享库的时候，会搜集每一个`..got`类型的重定位信息，然后构建 GOT，保证它含有每一个必须的入口通用变量也必须以这种方式被存取.

## 8.2.4 把符号导出给库用户.

如果你需要把符号导出给库用户，你必须把它们声明为函数或数据，如果它们是数据，你必须给出数据元素的 size.这是因为动态连接器必须为每一个导出的函数构建过程连接表入口，还要把导出数据元素从库的数据段中移出.所以，导出一个函数给库用户，你必须这样:

```nasm
       global  func:function           ; declare it as a function 
       
       func:   push    ebp 
       
               ; etc.
```

而导出一个数据元素，比如数组，你必须这样写代码:

```nasm
      global  array:data array.end-array      ; give the size too 
       
       array:  resd    128 
       .end:
```

小心:如果你希望通过把变量声明为`GLOBAL`并指定一个 size，而导出给库用户，这个变量最终会存在于主程序的数据段中，而不是在你的库的数据段内，所以你必须通过使用`..got`机制来获取你自己的全局变量，而不是`..gotogg`，就象它是一个外部变量一样(实际上，它已经变成了外部变量).同样的，如果你需要把一个导出的全局变量的地址存入你的一个数据段中，你不能通过下面的标准方式实现:

```nasm
       dataptr:        dd      global_data_item        ; WRONG
```

NASM 会以个普通的重定位解释这段代码，在这里，`global_data_item`仅仅是一个从`.data`段(或者其他段)开始的一个偏移值;

所以这个引用最终会指向你的数据段，而不是导出全局变量.对于上面的代码，你应该这样写:

```nasm
       dataptr:        dd      global_data_item wrt ..sym
```

这时使用了一个特殊的`WRT`类型`..sym`来指示 NASM 到符号表中去寻找一个在这个地址的特定符号，而不是通过段基址重定位.另外一种方式是针对函数的:以下面的方法引用你的一个函数:

```nasm
       funcptr:        dd      my_function

       will give the user the address of the code you wrote, whereas

       funcptr:        dd      my_function wrt .sym
```

会给出过程连接表中的该函数的地址，这是真正的调用程序应该得到的地址.两种地址都是可行的.

## 8.2.5 从库外调用过程.

从你的共享库外部调用过程必须通过使用过程连接表(PLT)才能实现，PLT 被放在库载入处的一个已知的偏移地址处，所以库代码可以以一种地址无关的方式去调用 PLT.在 PLT 中有跳转到含在 GOT 中的偏移地址的代码，所以对共享库中或主程序中的函数调用可以被转化为直接传递它们的真实地址.要调用一个外部过程，你必须使用另一个特殊的 PIC 重定位类型，`WRT ..plt`.这个比基于 GOT 的要简单得多:你只需要把调用`CALL printf`替换为 PLT 相关的版本:`CALL printf WRT ..plt`.

8.2.6 产生库文件.
======

写好了一些代码模块并把它们汇编成`.o`文件后，你就可以产生你的共享库了，使用下面的命令就可以:

```shell
$ ld -shared -o library.so module1.o module2.o       # for ELF 
$ ld -Bshareable -o library.so module1.o module2.o   # for BSD
```

对于 ELF，如果你的共享库要放在系统目录`/usr/lib`或`/lib`中，那对连接器使用`-soname`可以把最终的库文件名和版本号放进库中:

```shell
$ ld -shared -soname library.so.1 -o library.so.1.2 *.o
```

然后你就可以把`library.so.1.2`拷贝到库文件目录下，然后建立一个它的符号连的妆`library.so.1`.
