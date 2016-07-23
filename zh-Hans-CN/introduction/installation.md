1.3 安装
======

## 1.3.1 在 dos 和 Windows 下安装 NASM

如果你拿到了 NASM 的 DOS 安装包，`nasmXXX.zip`(这里.`XXX`表示该安装包的 NASM 版本号)，把它解压到它自己的目录下(比如:`c:\nasm`)

该包中会包含有四个可执行文件:NASM 可拟行文件`nasm.exe`和`nasmw.exe`，还有NDISASM 可执行文件`ndisasm.exe`和`ndisasmw.exe`。

文件名以`w`结尾的是`Win32`可执行格式。是运行在`Windows 95`或`Windows NT`的 Intel 处理器上的，另外的是16 位的`DOS`可执行文件。

NASM 运行时需要的唯一文件就是它自己的可执行文件，所以可以拷贝`nasm.exe`和`nasmw.exe`的其中一个到你自己的路径下，或者可以编写一个`autoexec.bat`把nasm 的路径加到你的`PATH`环境变量中去。(如果你只安装了 Win32 版本的，你可能希望把文件名改成`nasm.exe`。)就这样，NASM 装好了。

你不需要为了运行 nasm 而让`nasm`目录一直存在(非你把它加到了你的`PATH`中)，所以如果你需要节省空间，你可删掉它，但是，你可能需要保留文档或测试程序。

如果你下载了 DOS 版的源码包，`nasmXXXs.zip`，那`nasm`目录还会包含完整的 NASM 源代码，你可以选择一个 Makefiles 来重新构造你的 NASM 版本。

注意源文件`insnsa.c`， `insnsd.c`， `insnsi.h`和`insnsn.c`是由`standard.mac`中的指令自动生成的，尽管 NASM0.98 发布版中包含了这些产生的文件，你如果改动了insns.dat，standard.mac 或者文件，可能需要重新构造他们，在将来的源码发布中有可能将不再包含这些文件，多平台兼容的 Perl 可以从 www.cpan.org 上得到。

## 1.3.2 在 unix 下安装 NASM

如果你得到了 Unix 下的 NASM 源码包`nasm-x.xx.tar.gz`(这里 x.xx 表示该源码包中的nasm 的版本号)，把它解压压到一个目录，比如`/usr/local/src`。

包被解压后会创建自己的子目录`nasm-x.xx`NASM 是一个自动配置的安装包:一旦你解压了它，`cd`到它的目录下，输入`./configuer`，该脚本会找到最好的 C 编译器来构造 NASM，并据此建立 Makefiles。

一旦 NASM 被自动配置好后，你可以输入`make`来构造`nasm`和`ndisasm`二进制文件，然后输入`make install`把它们安装到`/usr/local/bin`，并把 man 页安装到`/usr/local/man/man1`下的`nasm.1 和`ndisasm.1`或者你可以给配置脚本一个`--prefix`选项来指定安装目录，或者也可以自己来安装。

NASM 还附带一套处理`RDOFF`目标文件格式的实用程序，它们在`rdoff`子目录下，你可以用`make rdf`来构造它们，并使用`make rdf_install`来安装。如果你需要的话。

如果 NASM 在自动配置的时候失败了，你还是可以使用文件`Makefile.unx`来编译它们，把这个文件改名为`Makefile`，然后输入`make`。

在`rdoff`子目录下同样有一个Makefile.unx 文件。