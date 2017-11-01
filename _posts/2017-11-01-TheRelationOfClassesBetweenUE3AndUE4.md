---
layout: post
title: UE3和UE4类型对应
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
| 其他待补充 | - |