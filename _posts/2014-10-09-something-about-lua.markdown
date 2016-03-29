---
layout: post
title: lua学习杂记
date: 2014-10-09
category: lua
tags: [lua, table, coroutine]
---
### 关于lua中table的两个常用API ###


1.`table.concat(table, [sep="", start=1, end=数组长度])` 

类似于python中字符串的join方法，举个例子，比如在lua中
{% highlight lua %}
tb = {"a", "b", "c"}
print("The concat result:"..table.concat(tb, ";"))
{% endhighlight %}
输出结果同python中这段代码是一样的，都是`a;b;c`
{% highlight python %}
tb = ["a", "b", "c"]
print ";".join(tb)
{% endhighlight %}

2.`table.insert(table, [pos,] value)`

pos默认为数组末尾。


----------

### lua中的字符串 ###

1.`string.format(format, str)`
类似于C中的printf，用于控制格式。
2.字符串长度
比起Python中还需要用len()函数获得字符长度，lua中意外地可以方便地使用警号来获得字符串长度。
{% highlight lua %}
a = "123"
print(a) --output: 123
print(#a) --output: 3
{% endhighlight %}


----------

### lua中的coroutine ###

这篇文章[【深入Lua】理解Lua中最强大的特性-coroutine（协程）][coroutine]讲得已经很详尽了，而且以云风大神的代码来进行了解释。不过那段代码的运行结果他写错了，在此更正一下。
{% highlight lua %}
function foo(a)
    print("foo", a)
    return coroutine.yield(2 * a)
end

co = coroutine.create(function ( a, b )
    print("co-body", a, b)
    local r = foo(a + 1)
    print("co-body", r)
    local r, s = coroutine.yield(a + b, a - b)
    print("co-body", r, s)
    return b, "end"
end)

print("main", coroutine.resume(co, 1, 10))
print("main", coroutine.resume(co, "r"))
print("main", coroutine.resume(co, "x", "y"))
print("main", coroutine.resume(co, "x", "y"))
{% endhighlight %}
实际的运行结果如下：

<img src="/assets/image/coroutine.png" />

原博文中错误的是第7行输出，应该为`main true 10 end`

[coroutine]:http://my.oschina.net/wangxuanyihaha/blog/186401