---
layout: post
title: Cocos2d个人项目：《晴天娃娃》Demo
date: 2016-12-12
category: Portfolio
tags: [c++, cocos2d]
---

* 目录
{:toc #hello}

# 简介
《晴天娃娃》是一款基于Cocos2d开发的iOS横版单机手游，玩家将操作晴天娃娃进行冒险，并通过改变天气来帮助晴天娃娃过关。途中，晴天娃娃遇见了一个和父亲走失的小女孩……

# 人物

#### 晴天娃娃
<img src="/img/in-post/portfolio/sunnydoll/doll_walk.gif" alt="晴天娃娃" width="10%" /><img src="/img/in-post/portfolio/sunnydoll/dollwithgirl.gif" alt="女孩" width="10%" /><img src="/img/in-post/portfolio/sunnydoll/doll_drown.gif" alt="晴天娃娃" width="10%" />
<img src="/img/in-post/portfolio/sunnydoll/dollwithgirl_drown.gif" alt="女孩" width="10%" />


#### 小女孩
<img src="/img/in-post/portfolio/sunnydoll/girl_walk.gif" alt="女孩" width="10%" />


# 游戏特性

1. **控制天气**：每一关卡都需要合适的天气触发相应的交互才有可能过关
2. **道具收集**：不同道具可以让晴天娃娃具有不同的功能
3. **多种因素影响结局走向**：道具的收集、天气的选择（未完成）等会影响结局分支

# 演示视频

<embed height="415" width="544" quality="high" allowfullscreen="true" type="application/x-shockwave-flash" src="http://static.hdslb.com/miniloader.swf" flashvars="aid=7381996&page=1" pluginspage="http://www.adobe.com/shockwave/download/download.cgi?P1_Prod_Version=ShockwaveFlash">

# 开发后记
1. 引擎的代码多多少少要去看或改，**万一引擎出Bug了呢**……
2. 坐标系、缩放比例、屏幕适配有些磨人，还需改进，可想而知**安卓开发者**的辛苦。
3. 学习了物理系统、粒子系统和帧动画，以后试试骨骼动画。
4. 感觉cocos2d封装的物理引擎通过**几串二进制码的或与**来判定刚体是否碰撞、刚体碰撞是否发出消息有点反人类，为何不做成像事件注册一样注册两类刚体的碰撞呢？
5. 画素材没有想象中那么难，但是画帧动画就没有想象中那么简单了。

