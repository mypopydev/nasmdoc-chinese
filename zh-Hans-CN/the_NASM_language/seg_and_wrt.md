3.6 `SEG`和`WRT`
======

当写很大的 16 位程序时，必须把它分成很多段，这时，引用段内一个符号的地址的能力是非常有必要的，NASM 提供了`SEG`操作符来实现这个功能。

`SEG`操作符返回符号所在的首选段的段基址，即一个段基址，当符号的偏移地址以它为参考时，是有效的，所以，代码:

```nasm
               mov     ax,seg symbol 
               mov     es,ax 
               mov     bx,symbol
```

总是在`ES:BX`中载入一个指向符号`symbol`的有效指针。

而事情往往可能比这还要复杂些:因为 16 位的段与组是可以相互重叠的，你通常可能需要通过不同的段基址，而不是首选的段基址来引用一个符号，NASM 可以让你这样做，通过使用`WRT`关键字，你可以这样写:

```nasm
               mov     ax,weird_seg        ; weird_seg is a segment base 
               mov     es,ax 
               mov     bx,symbol wrt weird_seg
```

会在`ES:BX`中载入一个不同的，但功能上却是相同的指向`symbol`的指针。

通过使用`call segment:offset`，NASM 提供 fall call(段内)和 jump，这里`segment`和`offset`都以立即数的形式出现。

所以要调用一个远过程，你可以如下编写代码:

```nasm
               call    (seg procedure):procedure 
               call    weird_seg:(procedure wrt weird_seg)
```

(上面的圆括号只是为了说明方便，实际使用中并不需要)NASM 支持形如`call far procedure`的语法，跟上面第一句是等价的。

`jmp`的工作方式跟`call`在这里完全相同。

在数据段中要声明一个指向数据元素的远指针，可以象下面这样写:

```nasm
               dw      symbol, seg symbol
```

NASM 没有提供更便利的写法，但你可以用宏自己建造一个。