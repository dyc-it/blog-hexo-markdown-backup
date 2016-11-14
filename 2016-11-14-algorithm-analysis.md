---
title: 算法分析
date: 2016-11-14 15:26:14
categories: algorithm
tags: algorithm
---

算法的时间复杂度和空间复杂度分析

<!-- more -->

# 时间复杂度
算法的时间复杂度是以基本语句执行次数的数量级来衡量的。

## 渐进记号
- 大O  
渐进上界，关注最坏情况下的复杂度
- 大Ω  
渐进下界
- 大Θ  
渐进确界

## 计算步骤
```
def increase_number(n):
    inc = 0
    for i in xrange(1, n):
        inc += 1
        for j in xrange(1, n):
            inc += 2
```

1. 找出算法中的基本语句  
算法中执行次数最多的那条语句就是基本语句，通常是最内层循环的循环体。比如上面代码中的"inc += 2" 
2. 计算基本语句执行次数的数量级  
基本执行次数表示为一个函数，然后去掉常熟系数和低次幂，比如表示为n<sup>2</sup>
3. 用大O记号表示算法的时间复杂度  
将基本执行次数的数量级放入大O标记中，上面算法的时间复杂度可以表示为O(n<sup>2</sup>)

## 常见时间复杂度
Ο(1)＜Ο(log<sub>2</sub>n)＜Ο(n)＜Ο(nlog<sub>2</sub>n)＜Ο(n<sup>2</sup>)＜Ο(n<sup>3</sup>)＜…＜Ο(2<sup>n</sup>)＜Ο(n!)  

- P类问题（Polynomial）  
Ο(1)、Ο(log<sub>2</sub>n)、Ο(n)、Ο(nlog<sub>2</sub>n)、Ο(n<sup>2</sup>)、Ο(n<sup>3</sup>)等称为多项式时间，被计算机科学家归类为P类问题
- NP类问题（Non-deterministic Polynomial）  
Ο(2<sup>n</sup>)、Ο(n!)称为指数时间，被计算机科学家归类为NP问题

## 例子




