---
title: 动态规划
date: 2016-11-15 18:43:56
categories: algorithm
tags: algorithm
---

动态规划介绍及应用

<!-- more -->
## 介绍
### 适用情况

1. 子问题的解会重复计算的情况：如果Fibonacci问题，将计算结果保存起来供以后使用
2. 


## 应用
### Fibonacci
- 普通版：存在大量的重复计算，时间复杂度为O(2<sup>n</sup>) 

```
def fibonacci_normal(n):
    if n == 0 or n == 1:
        return n
    else:
        return fibonacci_normal(n - 1) + fibonacci_normal(n - 2)
```

-  动态规划版：保存了中间结果，避免了重复计算，时间复杂度为O(n)

```
tmp_result = dict()
tmp_result[0] = 0
tmp_result[1] = 1


def fibonacci_dp(n):
    if n not in tmp_result:
        tmp_result[n] = fibonacci_dp(n - 1) + fibonacci_dp(n - 2)
    return tmp_result[n]
```
### 最长公共子序列

### 背包问题

### Floyd-Warshall算法

### Viterbi算法