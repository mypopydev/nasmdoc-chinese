5.2 `SECTION`或`SEGMENT`: 改变和定义段。
======

`SECTION`指令(`SEGMENT`跟它完全等效)改变你正编写的代码将被汇编进的段。

在某些目标文件格式中，段的数量与名称是确定的;

而在别一些格式中，用户可以建立任意多的段。

因此，如果你企图切换到一个不存在的段，`SECTION`有时可能会给出错误信息，或者定义出一个新段，Unix 的目标文件格式和`bin`目标文件格式，都支持标准的段`.text`，`.data`和`bss`段，与之不同的的，`obj`格式不能辩识上面的段名，并需要把段名开头的句点去掉。


## 5.2.1 宏 `__SECT__`

`SECTION`指令跟一般指令有所不同，的用户级形式跟它的原始形式在功能上有所不同，原始形式[SECTION xyz]，简单地切换到给出的目标段。

用户级形式，`SECTION xyz`先定义一个单行宏`__SECT__`，定义为原始形式[SECTION]，这正是要执行的指令，然后执行它。

所以，用户级指令:

```nasm
       SECTION .text
```

被展开成两行:

```nasm
       %define __SECT__        [SECTION .text] 
               [SECTION .text]
```

用户会发现在他们自己的宏中，这是非常有用的。

比如，4.3.3 中定义的宏`writefile`以下面的更为精致的写法会更有用:

```nasm
       %macro  writefile 2+ 
       
               [section .data] 
       
         %%str:        db      %2 
         %%endstr: 
       
               __SECT__ 
       
               mov     dx,%%str 
               mov     cx,%%endstr-%%str 
               mov     bx,%1 
               mov     ah,0x40 
               int     0x21 
       
       %endmacro
```

这个形式的宏，一次传递一个用出输出的字符串，先用原始形式的`SECTION`切换至临时的数据段，这样就不会破会宏`__SECT__`。

然后它把它的字符串声明在数据段中，然后调用`__SECT__`切换加用户先前所在的段。

这样就可以避免先前版本的`writefile`宏中的用来跳过数据的`JMP`指令，而且在一个更为复杂的格式模型中也不会失败，用户可以把这个宏放在任何独立的代码段中进行汇编。