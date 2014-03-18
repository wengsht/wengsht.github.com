--- 
layout: post
title: Function Tracer Using clang++ -- application and principle
abstract: This post will show you how to use clang++ for function tracing, and show how it work in assembly view.

tags: 
- Personal

type: post
---

![](/file/image/clang/clang.png)

## How to trace function call ##
Gcc and clang have a parameter to make the function tracer automatical, which is `-finstrument-functions`, you can `man g++` and check more information for it. Here I will first make a example to show how to use it fastly.

<hr/>

1\. Touch a file `main.cpp` and code:

{% highlight cpp linenos %}
#include <stdio.h>

void func() {
    printf("in func[%p]\n", func);
}

int main() {
    func();
    
    printf("in main[%p]\n", main);
            
    return 0;
}
{% endhighlight %}

<hr/>

2\. Touch a file `trace.cpp` and code:

It is important to put the function in a `extern "C"` declaration.

{% highlight cpp linenos %}
#include "stdio.h"
extern "C" {
    void __cyg_profile_func_enter(void *des, void *src_call) {
        printf("entering func %p from %p\n", des, src_call);
    }
    void __cyg_profile_func_exit(void *des, void *src_call) {
        printf("exiting func %p from %p\n", des, src_call);
    }
}
{% endhighlight %}

<hr/>

3\. build them with clang++

{% highlight bash linenos %}
$ clang++ main.cpp  -c -finstrument-functions  -o main.o
$ clang++ trace.cpp -c -o trace.o
$ clang++ main.o trace.o -o main && ./main

entering func 0x100b73e30 from 0x7fff8fa5f5fd
entering func 0x100b73dd0 from 0x100b73e57
in func[0x100b73dd0]
exiting func 0x100b73dd0 from 0x100b73e57
in main[0x100b73e30]
exiting func 0x100b73e30 from 0x7fff8fa5f5fd
{% endhighlight %}

You can see the function `__cyg_profile_func_enter` and its exit was called twice by function `func` and `main`.

<hr/>

## How did it work ##

It is easy to see how it work with disassembling tools, like `objdump`. You can `objdump -dS main | grep func -A 20`, which will output like following:

{% highlight bash linenos %}
...

0000000100000dd0 <__Z4funcv>:
   100000dd0:	55                   	push   %rbp
   100000dd1:	48 89 e5             	mov    %rsp,%rbp
   100000dd4:	48 83 ec 10          	sub    $0x10,%rsp
   100000dd8:	48 8d 05 f1 ff ff ff 	lea    -0xf(%rip),%rax        # 100000dd0 <__Z4funcv>
   100000ddf:	48 8b 75 08          	mov    0x8(%rbp),%rsi
   100000de3:	48 89 c7             	mov    %rax,%rdi
   100000de6:	e8 b5 00 00 00       	callq  100000ea0 <___cyg_profile_func_enter>
   100000deb:	48 8d 3d 40 01 00 00 	lea    0x140(%rip),%rdi        # 100000f32 <_printf$stub+0x20>
   100000df2:	48 8d 35 d7 ff ff ff 	lea    -0x29(%rip),%rsi        # 100000dd0 <__Z4funcv>
   100000df9:	b0 00                	mov    $0x0,%al
   100000dfb:	e8 12 01 00 00       	callq  100000f12 <_printf$stub>
   100000e00:	48 8d 35 c9 ff ff ff 	lea    -0x37(%rip),%rsi        # 100000dd0 <__Z4funcv>
   100000e07:	48 8b 7d 08          	mov    0x8(%rbp),%rdi
   100000e0b:	48 89 7d f8          	mov    %rdi,-0x8(%rbp)
   100000e0f:	48 89 f7             	mov    %rsi,%rdi
   100000e12:	48 8b 75 f8          	mov    -0x8(%rbp),%rsi
   100000e16:	89 45 f4             	mov    %eax,-0xc(%rbp)
   100000e19:	e8 c2 00 00 00       	callq  100000ee0 <___cyg_profile_func_exit>
   100000e1e:	48 83 c4 10          	add    $0x10,%rsp
   100000e22:	5d                   	pop    %rbp
   100000e23:	c3                   	retq
   100000e24:	66 66 66 2e 0f 1f 84 	data32 data32 nopw %cs:0x0(%rax,%rax,1)
   100000e2b:	00 00 00 00 00
   
...
{% endhighlight %}

You can see it just insert the `___cyg_profile_func_enter` after get into the function, and then insert `___cyg_profile_func_exit` call before `retq`.

What I am interesting for in the beginning is, how it works, when the return situation became complex with many `if`, `else`, `for` and `while`. 

To get the point, I just change the function `func` in `main.cpp` to more complex one like this:

{% highlight cpp linenos %}
int func(int x) {
    for(int i = 0;i < x;i++) {
        printf("in func[%p]\n", func);
        if(i > 50)
            return -1;
    }
    switch(x) {
        case 1:
            return 1;
        case 2:
            return -1;
    }
    return 0;
}
{% endhighlight %}

and What I get from the `objdump` is that one `___cyg_profile_func_exit` only before `retq`. That is say the compiler will push different return value into `%rax` and the jump to the `___cyg_profile_func_exit`. That is cool.

