---
layout: post
title:  "刘勇《推导实战模拟.xlsx》推导过程分析及公式说明"
date:   2020-06-25 10:02:25 +0800
categories: game_design
---

> 文件可以在[这里](https://bbs.gameres.com/forum.php?mod=viewthread&tid=798707&extra=page%3D1)下载

本文主要讲一下刘老师在这份文件里的推导过程，还有公式的说明，直接看Excel里的公式，不是很直观，总结成文字，更方便查看，也更容易理解思路。

# 1. 升级
#### 目的
推导出玩家每一等级升级所花的时长，为后面计算经验值做铺垫。

这里的时间，假设玩家纯打怪获得经验所花的升级时间，实际游戏中不可能纯钉怪，主要是任务、活动产出了，这里只是方便计算而做的假设。

#### 公式
```
X等级升级时间 = X-1等级升级时间 + X-1等级成长率
```

#### 说明
* 总共规划了 120 级
* 以下等级时，升级时间和成长率有变动（手动填写）：1、5、10、20、30、40、50、60、70、80、90、100、110、120
* 升级曲线是分段的，每一段的成长速度不同

# 2. 经验

#### 目的

#### 公式

#### 说明


