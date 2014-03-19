--- 
layout: post
title: Using LTTng For Function Tracing in Userspace
abstract: Mainly show how to using lttng for function tracing in userspace, beginning with a short introduce how to install lttng and use it. 

tags: 
- Personal

type: post
---

![](/file/image/lttng/lttng.png)

## Install LTTng for Userspace and Run Demo ##

<hr/>

To tracing work in userspace, you have 4 to install listing here:
1. [liburcu](http://git.lttng.org/?p=userspace-rcu.git;a=summary)
1. [LTTng-UST](http://git.lttng.org/?p=userspace-rcu.git;a=summary)
1. [LTTng-Tools](http://git.lttng.org/?p=lttng-tools.git)
1. [babeltrace](http://git.lttng.org/?p=userspace-rcu.git;a=summary)

For more informations how to get the source and build then, please the [LTTng Download Page](http://lttng.org/download).

The Readmes in the source tree is enough for installing.

Once you finish installing, you can try the demo in `path/to/lttng-ust/doc/expmples/demo`, its Readme has the steps trying it:

{% highlight bash linenos %}
$ lttng create demo -o ./out
$ lttng enable-event -u -a -c chan1
$ lttng start
$ ./demo-trace
$ lttng destroy
{% endhighlight %}

For reports reading, you have to use the tool `babeltrace`:

{% highlight bash linenos %}
$ babeltrace ./out

[09:47:52.307813098] (+?.?????????) wengsht.lab501 ust_tests_demo:starting: { cpu_id = 2 }, { value = 123 }
[09:47:52.307826726] (+0.000013628) wengsht.lab501 ust_tests_demo2:loop: { cpu_id = 2 }, { intfield = 0, intfield2 = 0x0, longfield = 0, netintfield = 0, netintfieldhex = 0x0, arrfield1 = [ [0] = 1, [1] = 2, [2] = 3 ], arrfield2 = "test", _seqfield1_length = 4, seqfield1 = [ [0] = 116, [1] = 101, [2] = 115, [3] = 116 ], _seqfield2_length = 4, seqfield2 = "test", stringfield = "test", floatfield = 2222, doublefield = 2 }

...

{% endhighlight %}

It's easy to know how the trace work by reading demo.c, you don't have to deep into the source tree...

## Function Tracer Using LTTng ##
<hr/>

[Lttng-ust-cyg](http://lttng.org/files/doc/man-pages/man3/lttng-ust-cyg-profile.3.html) is a lttng function tracer base on `-finstrument-functions` in `gcc` or `clang`. For more information about this parameter, you can see my [last blog post](http://wengsht.github.io/2014/03/16/Function%2BTracer%2BUsing%2Bclang%2B%2B%2B--%2Bapplication%2Band%2Bprinciple.html).

1\. Touch a file `main.c` and code:
{% highlight c linenos %}
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

2\. Build it with `gcc` or `clang`:

{% highlight bash linenos %}
clang main.c -o main -finstrument-functions
{% endhighlight %}


3\. Touch a bash file `trace.sh` and code:

{% highlight bash linenos %}
#!/bin/sh
lttng create demo -o ./out
lttng enable-event -u -a -c chan1
lttng start
LD_PRELOAD=liblttng-ust-cyg-profile-fast.so ./main
lttng destroy
{% endhighlight %}

4\. Run the bash and `babeltrace`:

{% highlight bash linenos %}
$ sh trace.sh

Session demo created.
Traces will be written in /home/wengsht/program/test/out
All UST events are enabled in channel chan1
Tracing started for session demo
in func[0x4005f4]
in main[0x400633]
Session demo destroyed

$ babeltrace ./out
[11:16:30.679528311] (+?.?????????) wengsht.lab501 lttng_ust_cyg_profile_fast:func_entry: { cpu_id = 2 }, { addr = 0x400633 }
[11:16:30.679533894] (+0.000005583) wengsht.lab501 lttng_ust_cyg_profile_fast:func_entry: { cpu_id = 2 }, { addr = 0x4005F4 }
[11:16:30.679549541] (+0.000015647) wengsht.lab501 lttng_ust_cyg_profile_fast:func_exit: { cpu_id = 2 }, { }
[11:16:30.679551325] (+0.000001784) wengsht.lab501 lttng_ust_cyg_profile_fast:func_exit: { cpu_id = 2 }, { }
{% endhighlight %}

You can see the tracer have got into the `main(0x400633)` and `func(0x4005f4)`. It seems that there is not a filter to use because of the parameter basis. It must be a useful work to something like filter work. 
