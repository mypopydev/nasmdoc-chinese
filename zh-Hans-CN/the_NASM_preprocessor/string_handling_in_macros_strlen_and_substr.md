4.2 字符串处理宏: `%strlen` and `%substr`
======

在宏里可以处理字符串通常是非常有用的。

NASM 支持两个简单的字符串处理宏，通过它们，可以创建更为复杂的操作符。

## 4.2.1 求字符串长度: `%strlen`

`%strlen`宏就像`%assign`，会为宏创建一个数值型的值。

不同点在于`%strlen`创建的数值是一个字符串的长度。

下面是一个使用的例子:c

```nasm
       %strlen charcnt 'my string'
```

在这个例子中，`charcnt`会接受一个值 8，就跟使用了`%assign`一样的效果。

在这个例子中，`my string`是一个字面上的字符串，但它也可以是一个可以被扩展成字符串的单行宏，就像下面的例子:

```nasm
       %define sometext 'my string' 
       %strlen charcnt sometext
```

就像第一种情况那样，这也会给`charcnt`赋值 8

## 4.2.2 取子字符串: `%substr`

字符串中的单个字符可以通过使用`%substr`提取出来。

关于它使用的一个例子可能比下面的描述更为有用:%substr mychar `xyz` 1%substr mychar `xyz` 2%substr mychar `xyz` 3;

```nasm
       %substr mychar  'xyz' 1         ; equivalent to %define mychar 'x' 
       %substr mychar  'xyz' 2         ; equivalent to %define mychar 'y' 
       %substr mychar  'xyz' 3         ; equivalent to %define mychar 'z'
```

在这个例子中，mychar 得到了值`z`。

就像在`%strlen`(参阅 4.2.1)中那样，第一个参数是一个将要被创建的单行宏，第二个是字符串，第三个参数指定哪一个字符将被选出。

注意，第一个索引值是 1 而不是 0，而最后一个索引值等同于`%strlen`给出的值。

如果索引值超出了范围，会得到一个空字符串。