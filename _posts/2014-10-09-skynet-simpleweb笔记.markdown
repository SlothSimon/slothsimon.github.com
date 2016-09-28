---
layout: post
title: skynet:simpleweb笔记
date: 2014-10-09
categories: Notes
tags: [skynet, network, lua]
---
# simpleweb函数追溯 #
simpleweb中有一个本地的response函数用以返回客户端传来的request。

{% highlight lua %}
local function response(id, ...)
	local ok, err = httpd.write_response(sockethelper.writefunc(id), ...)
	if not ok then
		-- if err == sockethelper.socket_error , that means socket closed.
		skynet.error(string.format("fd = %d, %s", id, err))
	end
end
{% endhighlight %}

该response可以在客户端的页面（比如浏览器界面）打印出request内容，也就是纯粹地镜像返回request内容。

<img src="/assets/image/simpleweb.png" />

打印的这些内容是在simpleweb第21行的函数内组织成所要的table格式，然后传递给response。
response-->httpd.write_response-->(**httpd** local)writeall-->sockethelper.writefunc（该函数返回一个由fd/id生成的未调用函数..(content)用以在某个具体socket中写入content）-->writebytes-->socket.write-->(前缀全局名实际为socketdriver)driver.send-->socketdriver.so

关于动态链接库还不懂，具体需要阅读http://blog.sina.com.cn/s/blog_62ef2f1401014r9s.html

总之目前先认为是一个在屏幕上打印内容的函数吧。也就是所谓浏览器的"print"API
write_response函数中可以传自定义的bodyfunc()函数用以输出body，还不知道怎么具体应用。总之传过去的内容只能是string, function或者nil，否则会抛出错误。

writeall()中response的header信息和网页的html信息都是用writefunc输出，但是似乎socketdriver.so中的接口会自动判断哪些是response的header信息哪些是输出到网页页面上的信息。不过在我胡乱给了个table作为header参数时，本来应该在response header中的"content-length: 420"出现在了网页页面上，不知道这算不算bug。

测试simpleweb时发现每一次刷新或打开页面都会提交两次request，第一次请求的内容是/，也就是用户需要的网页，第二次是favicon.ico。这是由于第一次请求的网页meta标签中没有设定ico，浏览器就会自动提交一个请求网站图标的request。具体见https://cnodejs.org/topic/54056647e84941a5714d65f4

skynet.newservice --> skynet.call(".launcher", 'lua', 'LAUNCH', 'snlua', name, ...)-->skynet.call(addr, typename, ...)--> skynet.core.send(addr, p.id, nil, p.pack(...)) 和 yield_call(service, session)

接下来找不到skynet.core这个函数库，不明 ；以及这里使用的coroutine.resume和coroutine.yield采用的是云风自己重新写的一个。

关于这部分函数的说明摘取[skynet的wiki][wiki]
> skynet.call(address, typename, ...) 这条 API 则不同，它会在内部生成一个唯一 session ，并向 address 提起请求，并阻塞等待对 session 的回应（可以不由 address 回应）。当消息回应后，还会通过之前注册的 unpack 函数解包。表面上看起来，就是发起了一次 RPC ，并阻塞等待回应。


> skynet.newservice(name, ...) 用于启动一个新的 Lua 服务。name 是脚本的名字（不用写 .lua 后缀）。只有被启动的脚本的 start 函数返回后，这个 API 才会返回启动的服务的地址，这是一个阻塞 API 。如果被启动的脚本在初始化环节抛出异常，或在初始化完成前就调用 skynet.exit 退出，｀skynet.newservice` 都会抛出异常。如果被启动的脚本的 start 函数是一个永不结束的循环，那么 newservice 也会被永远阻塞住。

[wiki]:https://github.com/cloudwu/skynet/wiki/LuaAPI
