---
layout: post
title: FObjectIterator源码笔记
date: 2017-11-07
category: notes
tags: [unreal4, unreal3, c++]
---


UE中使用的迭代器中有`TObjectIterator`和`FObjectIterator`，前者继承自后者，加诸了模板特性。

文中虽然用的UE4代码（因为其开源），但UE3的这个迭代器原理是一致的，而UE4稍加改进。

先从`FObjectIterator`注释看起。

>/**
> * Class for iterating through all objects, including class default objects.
> * Note that when Playing In Editor, this will find objects in the
> * editor as well as the PIE world, in an indeterminate order.
> */

意即：

> 这个类会迭代所有对象，包括默认类对象。需要注意的是当在编辑器中运行游戏时，不仅会找到游戏中的对象还会找到编辑器中的对象，顺序不确定。

而`TObjectIterator`则是
> 这个类用于迭代继承自给定类型的所有对象，不包含默认类对象。

构造函数：
```c++
/**
	 * Constructor
	 *
	 * @param	InClass						返回该类的对象或子类的对象
	 * @param	bOnlyGCedObjects			若为真，跳过所有永久对象
	 * @param	AdditionalExclusionFlags	具有对应RF_* flags的对象应被排除
	 */
	FObjectIterator(UClass* InClass = UObject::StaticClass(), bool bOnlyGCedObjects = false, EObjectFlags AdditionalExclusionFlags = RF_NoFlags, EInternalObjectFlags InInternalExclusionFlags = EInternalObjectFlags::None) 
		: FUObjectArray::TIterator(GUObjectArray, bOnlyGCedObjects)
		, Class(InClass)
		, ExclusionFlags(AdditionalExclusionFlags)
		, InternalExclusionFlags(InInternalExclusionFlags)
	{
		// 我们不会返回后台正在加载的对象，除非是在异步加载中。

		InternalExclusionFlags |= EInternalObjectFlags::Unreachable;
		if (!IsInAsyncLoadingThread())
		{
			InternalExclusionFlags |= EInternalObjectFlags::AsyncLoading;
		}
		check(Class);

		do
		{
			UObject *Object = **this;       // 先将迭代器指针变成迭代器，再变成对象指针。

			if (!(Object->HasAnyFlags(ExclusionFlags) || Object->HasAnyInternalFlags(InternalExclusionFlags) || (Class != UObject::StaticClass() && !Object->IsA(Class))))
			{
				break;
			}
		} while(Advance());
	}
```

其中，`InternalExclusionFlags |= EInternalObjectFlags::AsyncLoading`有些费解，毕竟从上下文看来，既然都不异步加载了，还有必要排除异步加载的对象吗？

再看`IsInAsyncLoadingThread()`和`EInternalObjectFlags::AsyncLoading`的定义。

```c++
/** @return True if called from the async loading thread if it's enabled, otherwise if called from game thread while is async loading code. */
extern CORE_API bool (*IsInAsyncLoadingThread)();

enum class EInternalObjectFlags : int32
{
    ...
    AsyncLoading = 1 << 27, ///< Object is being asynchronously loaded.		
    ...
};
```

于是明白，`IsInAsyncLoadingThread()`指的是当前代码运行所在线程是否异步，而不是指整个程序都不是异步加载。
根据[UDK官方文档](https://docs.unrealengine.com/udk/Three/ContentStreamingCH.html)，
>在异步加载过程中创建的对象将被标记为`RF_AsyncLoading`，因为直到加载完成之前，对于引擎的其它部分来说，它们处于隐藏状态。

也就是说，在游戏运行过程中，有主线程和其他线程，其中包含了异步加载的线程；游戏中存在着加载完毕的对象、异步加载中的对象等。

对于非异步加载的线程来说，异步加载中的对象是**隐藏的，不应该被获取的**，所以`InternalExclusionFlags`要加上`AsyncLoading`，迭代时碰到有此Flag的对象就跳过；而对于异步加载的线程来说，异步加载中的对象是**可见的，可以被获取进行操作**。

表面的原理了解了，但背后为什么这么设计尚不清楚，还需要对UE引擎的异步加载进行深入学习。

另外，UE3中`AdditionalExclusionFlags`和`InInternalExclusionFlags`是一个变量，UE4为什么将其分为两个部分，尚未可知。

值得吐槽的是，UE3这里的代码写的极其精炼，只用一行`while();`就完成了UE4中`do{}while()`的工作，复杂的while条件愣是让我看了好一会儿才理解。**不得不说，代码的小而精炼和易读性通常是不可兼得。作为一个开源项目，此时易读性更为重要。**

最后，提一些此类型代码细节：
1. `FObjectIterator`作为一个经常被使用的工具，它的实现会影响性能，因此其中很多函数都加了FORCEINLINE标识符
2. UE3用`++Index`而UE4用`Advance()`替代了，与时俱进跟随C++11新标准。
3. `check()`在development模式下运行，而`checkSlow`只在debug模式下运行。[[信息来源]](https://forums.unrealengine.com/development-discussion/c-gameplay-programming/14856-check-and-checkslow)