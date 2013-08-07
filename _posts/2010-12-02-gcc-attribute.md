---
layout: default
title: gcc 属性 section 在应用层代码的使用
---

Linux 内核 init/main.c 里的 do_initcalls 就是利用gcc的section属性来自动遍历
执行各个模块的初始化代码，从而避免复杂的注册处理。那么，在应用层代码里能否
借用这个方法呢，看下面的代码：

{% highlight c linenos %}
#include <stdio.h>
#include <string.h>

typedef void(*func)();

void func_a()
{
    printf("%s:%d\n", __func__, __LINE__);
}

 void func_b()
 {
     printf("%s:%d\n", __func__, __LINE__);
 }

 static func * const fn_a __attribute__((used, section("init"))) = (func * const)&func_a;
 static func * const fn_b __attribute__((used, section("init"))) = (func * const)&func_b;

 int main()
 {
     extern const func __start_init;
     extern const func __stop_init;

     func *f = (func *)&__start_init;

     while (f < (func *)&__stop_init) {
         (*f++)();
     }

     return 0;
 }
{% endhighlight %}

<pre>
# gcc -Wall -O2 a.c &amp;&amp; ./a.out
func_a:8
func_b:13
</pre>

可以看到，func_a 和 func_b被执行了。这个程序里的\_\_start_init 和\_\_stop_init 其
实是由链接器生成的，规则是在section名前加上 \_\_start_和\_\_stop_。

注意属性section前的used属性：在这个例子中如果打开优化会出现编译错误，原
因是fn_a和fn_b由于没有在任何其他地方用到而被优化掉。

<del>不过在打开优化选项时该程序编译不通过，还需要进一步研究。
<pre>
$ gcc -Wall -O1 a.c &amp;&amp; ./a.out
/tmp/ccvxshH3.o: In function `main':
a.c:(.text+0x7): undefined reference to `__start_init'
a.c:(.text+0xc): undefined reference to `__stop_init'
collect2: ld 返回 1
</pre></del>


另外采用这种方法执行的函数必须没有依赖关系，尤其对不同源文件中的函数，其执行顺序由链接器来决定。

{{ page.date | date_to_string }}
