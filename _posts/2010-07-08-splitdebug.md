---
layout: default
title: 实现ELF可执行文件和调试符号表分离
---

记得好久以前看到RedHat的rpm都有个 debuginfo 的对应rpm，也就是可执行文件没有
调试符号表，但安装debuginfo的rpm后可以看到调试信息。当时没搞明白是怎么做的，
后来就不了了之了。

前不久在Gentoo里看到FEATURES里加个splitdebug可以实现同样的效果，正好我们可能
有类似的需求，于是研究了一下，其实很简单的几个命令：

<pre>
$ gcc -g test.c -o test
$ objcopy --only-keep-debug test test.debug
$ objcopy --add-gnu-debuglink=test.debug test
$ stip -s test
$ gdb ./test
...
Reading symbols from /home/liuzx/test...Reading symbols from /home/liuzx/test.debug...done.
(no debugging symbols found)...done.
(gdb) list
1 #include &lt;stdio.h&gt;
2
3 int main()
4 {
5     printf("Hello.\n");
6
7     return 0;
8 }
(gdb) q
$ file test
test: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.9, stripped
</pre>

{{ page.date | date_to_string }}
