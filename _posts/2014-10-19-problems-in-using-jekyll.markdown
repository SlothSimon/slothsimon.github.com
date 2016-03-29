---
layout: post
title: jekyll使用中的问题
date: 2014-10-19
category: web development
tags: jekyll 
---
在编辑两篇在10月9日写好的两篇笔记时，调用`jekyll server`，先后出现了以下问题。

1.`undefined method 'default_proc'`

Google了半天，才发现原来是博文头信息里，冒号后面必须空一格，否则就会报错，真是\_(:3」∠)\_

错误：

		---
    	title:lua学习笔记
    	---
正确：

    	---
    	title: lua学习笔记
    	---

2.`warning: cannot close fd before spawn 'which' 不是内部或外部命令，也不是可运行的程序或批处理文件`

google了一把说是pygments的问题。我之前在`_config.yml`已经写明了`highlighter: pygments`，抱着死马当成活马的态度，改成了`highlighter: rouge/pygments`。结果问题就神秘消失了……但是这样会导致高亮出现问题，代码全是黑色，没有其他颜色。

3.`Please add the following to Gemfile to avoid polling for changes:
require 'rbconfig'
if RbConfig::CONFIG['target_os'] =~ /mswin|mingw|cygwin/i
	gem 'wdm', '>=0.1.0'
end`

最后在stackoverflow中找到类似问题[Ruby error: cannot load such file — wdm (LoadError)][wdm],只要安装wdm这个模块似乎就可以了。代码如下：

`gem install wdm`

4.中文乱码

其实以上问题（包括当前这个）在我写第二篇博文之前都没有发生过，不知道为什么新建了博文后，出现各种各样的问题，而且打开的网页中如果有中文还是乱码。但是我的第一篇博文也是中文，却没有任何显示上的问题，我在想是不是文本编码不同，但是都是 UTF-8编码，不知道问题出在哪里。
以及发现，原来`_post`文件夹中的文件名也可以包含中文没问题了。

补充：后来发现可能是format-matter中的内容写错了导致乱码，比如说只有一个分类的应写：

`category: lua`

有两个分类应写：

`categories: [lua, skynet]`

tags也是同样，不过tags不用改成tag。


[wdm]:http://stackoverflow.com/questions/20459859/ruby-error-cannot-load-such-file-wdm-loaderror