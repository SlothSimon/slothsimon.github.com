---
layout: post
title: CullDistanceVolume源码笔记
date: 2017-11-08
category: notes
tags: [unreal4, unreal3, c++]
---

## Introduction
`CullDistanceVolume`的用法是设置多个`CullDistanceSizePair`，在其内部的物体根据自身大小决定在多近的距离内才显示出来，超出该距离就不绘制在玩家画面上。

`CullDistanceSizePair`可以当做一个极简的Map成员来看待，比如Size=100的物体，可以查到其Distance=1000，那么玩家距离该物体1000米内才能看见它，超出1000米就不见了。形状复杂的物体，其Size由其边界Bound决定，如何计算在此就不展开了。

这样做的好处是，**灵活地节约性能**。

譬如一把Size=100的小椅子距离你有10000UU，那根本没必要渲染它，反正玩家也不会注意，或者根本看不见，并不影响玩家的游玩；但一座远景的大山，Size=8000，作为设计师你希望让玩家在很远的地方就能看见，那么就可以针对这个Size设置CullDistance。

当然了，有人会问那山上有座Size=100的小宝塔，我也希望玩家在很远的地方就能看见呢？那就得设置宝塔的`PrimitiveComponent`，其中有属性`MaxDrawDistance`和`AllowCullDistanceVolume`，勾选后者，然后`MaxDrawDistance`设为较大的值，就可以实现这个功能。

（若不勾选`AllowCullDistanceVolume`则取`MaxDrawDistance`和`CullDistanceVolume`中较小的值）

## Source Code

`CullDistanceVolume`的继承结构是这样，`Actor-->Brush-->Volume-->CullDistanceVolume`。

诸多属性中只有两个是新定义的，而方法也只有两个是在游戏运行过程中会用到的：
```c++
UCLASS(hidecategories=(Advanced, Attachment, Collision, Volume))
class ACullDistanceVolume
	: public AVolume
{
	GENERATED_UCLASS_BODY()
	
	/**
	 * Array of size and cull distance pairs. The code will calculate the sphere diameter of a primitive's BB and look for a best
	 * fit in this array to determine which cull distance to use.
	 */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category=CullDistanceVolume)
	TArray<struct FCullDistanceSizePair> CullDistances;

	/**
	 * Whether the volume is currently enabled or not.
	 */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category=CullDistanceVolume)
	uint32 bEnabled:1;
	
	public:
	
	... // Editor functions
	

	/**
	 * Returns whether the passed in primitive can be affected by cull distance volumes.
	 *
	 * @param	PrimitiveComponent	Component to test
	 * @return	true if tested component can be affected, false otherwise
	 */
	static bool CanBeAffectedByVolumes( UPrimitiveComponent* PrimitiveComponent );

	/** 
	  * Get the set of primitives and new max draw distances defined by this volume. 
	  * Presumes only primitives that can be affected by volumes are being passed in.
	  */
	void GetPrimitiveMaxDrawDistances(TMap<UPrimitiveComponent*,float>& OutCullDistances);
```

两个属性很简单，不介绍了。两个方法则都是在`UpdateCullDistanceVolume()`时使用到。

### CanBeAffectedByVolumes()
返回True的条件为
```
组件有非空Owner && 组件是Static && 组件允许CullDistanceVolume && 组件可见 && 组件不是模板 && 组件Owner所在World不为空 && 组件所在Scene是当前的Scene
```

### GetPrimitiveMaxDrawDistances()
这里主要是处理**多个地方**设置了Distance后，究竟该以哪个Distance为最终的`CullDistance`。
1. 找到`CullDistances`中最接近该物体大小的成员，然后取其Distance赋给`CurrentCullDistance`
2. 若该物体`CullDistance` > 0，则重设物体`CullDistance=Min(CullDistance, CurrentCullDistance)`；否则，直接设为`CurrentCullDistance`。

## 和UE3的对比
相比UE3，UE4优化了一些细节。

- 只在编辑器用到的函数通过宏来控制是否编译
- World变量在UE3中是通过全局变量`UWorld * GWorld`来存储和获得使用，`CanBeAffectedByVolumes()`并没有判断GWorld是否为空；但在UE4中是用`UWorldProxy GWorld`来存储，而获取则是通过`Actor->GetWorld()`来实现。
    - 猜测这样的改进是因为，UE4中游戏时（不考虑PIE），**有可能同时存在多个World**，譬如UT4中的即时回放就是通过新建另一个World来实现的。
- UE3中`Actor`有属性`bNoDelete`，在`CanBeAffectedByVolumes()`中与物体是否Static取或，但在UE4中没有该属性。
- UE3中`GetPrimitiveMaxDrawDistances()`是对所有`UPrimitiveComponent`迭代，然后在`OutCullDistances`中查找是否有该组件；而UE4则直接迭代`OutCullDistances`中所有组件，降低了复杂度。

## Review
考虑到这两个函数的用法都是和`UWolrd::UpdateCullDistanceVolume()`有关，下节就学习一下这个函数的内容。