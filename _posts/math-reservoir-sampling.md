---
layout: page
title:	蓄水抽样
category: blog
description: 
---
# Preface


果壳网中[google面试题](http://www.guokr.com/article/61878/?page=6)中的一道.
>随机选取一个长度为N的链表（N很大）里的K个元素 - 编程珠玑
给你一个长度为N的链表。N很大，但你不知道N有多大。你的任务是从这N个元素中随机取出k个元素。你只能遍历这个链表一次。你的算法必须保证取出的元素恰好有k个，且它们是完全随机的（出现概率均等）。

想了半天想的解法如下(其实这种解法就叫蓄水抽样)

>先提取前k个的元素到数组arr里面.
继续遍历链表,每遍历1个元素后,都与数组里面的值作一次是否留下的概率竞争.

![](/img/math.reservoir-sampling.png)
