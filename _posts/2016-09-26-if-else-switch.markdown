---
layout: post
title: if-else与switch的选择
date: 2016-09-28
category: Notes
tags: [c++]
---

### 参考资料：

- [switch 与 if else 效率分析与总结](http://blog.csdn.net/jmppok/article/details/17138325?locationNum=2)
- [Is “else if” faster than “switch() case”?](http://stackoverflow.com/questions/767821/is-else-if-faster-than-switch-case)
- [Is there any significant difference between using if/else and switch-case in C#?](http://stackoverflow.com/questions/395618/is-there-any-significant-difference-between-using-if-else-and-switch-case-in-c)

### 总结：

| code              | 适用场景                                                                      | 搜索方法          |
| -------------     |:-------------:                                                              | :-----:         |
| if-else           | 对于概率不同的条件，按从大到小的概率排列提高效率；适合条件数目<=5；适用于一切类型          | 顺序              |
| switch            | 无法确定概率优先的条件；可读性好；适合条件数目<=5；仅适用于int、char、enum class等类型   |   二分法或跳转表  |
| (unordered) map   | 简洁；适合条件数目>5；适用于一切类型                                               |    (hash)红黑树  |


表虽然这么列了，其实还有很多**tricky**的细节，比如有的编译器对于类似switch的if-else也会优化搜索，<font color='red'>所以99.99%（或者说100%...）情况下根本不用考虑二者的效率区别</font>，哪个写起来可读性好或者维护性好就可以了。优化这二者的工作应该由编译器完成。

除此之外，也要考虑具体的应用场景。

考虑用switch还是if-else是我写游戏状态机时遇到的问题，因为if-else虽然可以满足跳转条件，但是读起来实在没有switch那么一目了然，还要重复写v == 1, v == 2, ……

而switch的缺点呢，则是**不能用string作判断条件**，这就很蛋疼了，因为我用的是cocos2d的CustomEventName作为参数传递的。

对格式强迫症的我，最后还是选了switch，为此特地写了**enum与string之间的相互转换**，结果还是会用到map和数组。关于 enum的话题又可以另外写一篇日志了。

也许将来会换成map<string, void*>吧，然后把函数指针存在里面进行调用。
