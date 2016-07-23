5.5 `GLOBAL`: 把符号导出到其他模块中。
======

`GLOBAL`是`EXTERN`的对立面:如果一个模块声明一个`EXTERN`的符号，然后引用它，然后为了防止链接错误，另外某一个模块必须确实定义了该符号，然后把它声明为`GLOBAL`，有些汇编器使用名字`PUBLIC`。

`GLOBAL`操作符所作用的符号必须在`GLOBAL`之后进行定义。

`GLOBAL`使用跟`EXTERN`相同的语法，除了它所引用的符号必须在同一样模块中已经被定义过了，比如:

```nasm
       global _main 
       _main: 
               ; some code
```

就像`EXTERN`一样，`GLOBAL`允许目标格式文件通过冒号定义它们自己的扩展。

比如`elf`目标文件格式可以让你指定全局数据是函数或数据。

```nasm
       global  hashlookup:function, hashtable:data
```

就象`EXTERN`一样，原始形式的`GLOBAL`跟用户级的形式不同，仅能一次带有一个参数