5.6 `COMMON`: 定义通用数据域。
======

`COMMON`操作符被用来声明通用变量。

一个通用变量很象一个在非初始化数据段中定义的全局变量。

所以:

```nasm
       common  intvar  4
```

功能上跟下面的代码相似:

```nasm
       global  intvar 
       section .bss 
       
       intvar  resd    1
```

不同点是如果多于一个的模块定义了相同的通用变量，在链接时，这些通用变量会被合并，然后，所有模块中的所有的对`intvar`的引用会指向同一片内存。

就角`GLOBAL`和`EXTERN`，`COMMON`支持目标文件特定的扩展。

比如，`obj`文件格式允许通用变量为 NEAR 或 FAR，而`elf`格式允许你指定通用变量的对齐需要。

```nasm
       common  commvar  4:near  ; works in OBJ 
       common  intarray 100:4   ; works in ELF: 4 byte aligned
```

它的原始形式也只能带有一个参数。