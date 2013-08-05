---
layout: default
title: gcc 属性 section 在应用层代码的使用
---

Linux 内核 init/main.c 里的 do_initcalls 就是利用gcc的section属性来自动遍历
执行各个模块的初始化代码，从而避免复杂的注册处理。那么，在应用层代码里能否
借用这个方法呢，看下面的代码：

<pre>
1 #include &lt;stdio.h&gt;
2 #include &lt;string.h&gt;
3
4 typedef void(*func)();
5
6 void func_a()
7 {
8     printf("%s:%d\n", __func__, __LINE__);
9 }
10
11 void func_b()
12 {
13     printf("%s:%d\n", __func__, __LINE__);
14 }
15
16 static func * const fn_a __attribute__((used, section("init"))) = (func * const)&amp;func_a;
17 static func * const fn_b __attribute__((used, section("init"))) = (func * const)&amp;func_b;
18
19 int main()
20 {
21     extern const func __start_init;
22     extern const func __stop_init;
23
24     func *f = (func *)&amp;__start_init;
25
26     while (f &lt; (func *)&amp;__stop_init) {
27         (*f++)();
28     }
29
30     return 0;
31 }

# gcc -Wall -O2 a.c &amp;&amp; ./a.out
func_a:8
func_b:13
</pre>

可以看到，func_a 和 func_b被执行了。这个程序里的__start_init 和 __stop_init 其
实是由链接器生成的，规则是在section名前加上 __start_和__stop_。

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
