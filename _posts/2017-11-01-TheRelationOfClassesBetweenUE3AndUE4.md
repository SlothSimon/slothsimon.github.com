---
layout: post
title: UE3和UE4类型、概念对应
date: 2017-11-01
category: notes
tags: [unreal4, unreal3]
---

| UE3       | UE4       |  
| :-----:   | :-----:   |   
| Object,Actor,Pawn,Controller,World,Level,Component,Player等    | 一致    |  
| WorldInfo | WorldSetting |
| GameInfo  | GameMode(or GameModeBase)     |
| GameReplicationInfo | GameState |
| PlayerReplicationInfo | PlayerState |
| - | Character |
| Matinee | Sequencer(有Matinee但是已不更新逐渐在弃置) |
| AnimTree | Animation Blueprint |
| UnrealScript | 一部分并入C++，一部分和Kismet合并成为Blueprint，状态机则由BehaviorTree代替 |
| Kismet    | Blueprint，从源代码和蓝图方法来看，很多函数都有Kismet原型 |
| 其他待补充 | - |

总体而言，降低了对程序员的依赖，不写代码来完成一个游戏是完全有可能的。