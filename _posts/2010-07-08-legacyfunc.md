---
layout: default
title: 怎样让函数在被编译或被链接时警告用户？
---

使用了 mktemp 的程序在不是特别老的 libc 环境下编译时都会有类似的警告：

*warning: the use of \`mktemp' is dangerous, better use \`mkstemp'*

这个警告是在链接时产生的，实际上是GNU 连接器的一个扩展，对函数库中的某个函数
如果过时了可以采用这种方式来提醒用户。此外，在编译时也可产生相应的警告信息。
具体实现见下面的代码例子：

<pre>
$ more test.c
#include &lt;stdlib.h&gt;

void __attribute__((warning("use of dummy is obsoleted"))) dummy1(void)
{
    return;
}

int main()
{
    char template[] = "/tmp/test_XXXXXX";

    mktemp(template);
    dummy1();
    dummy2();

    return 0;
}
$ more lib.c
#define link_warning(symbol, msg)                             \
    static const char __evoke_link_warning_##symbol[]         \
    __attribute__ ((used, section (".gnu.warning." #symbol))) \
    = msg;

void dummy2(void)
{
    return;
}

link_warning(dummy2, "use of dummy2 is obsoleted");
$ gcc test.c lib.c
test.c: In function 'main':
test.c:13: warning: call to 'dummy1' declared with attribute warning: use of dummy is obsoleted
/tmp/.private/liuzx/ccyBD0XN.o: In function `main':
test.c:(.text+0x45): warning: use of dummy2 is obsoleted
test.c:(.text+0x36): warning: the use of `mktemp' is dangerous, better use `mkstemp'
</pre>

{{ page.date | date_to_string }}
