---
title: 动态规划
date: 2016-11-15 18:43:56
categories: algorithm
tags: algorithm
---

动态规划介绍及应用

<!-- more -->
# 简介

动态规划具有多项式时间的复杂度。  

## 适用情况

1. 子问题的解会重复计算的情况：如果Fibonacci问题，将计算结果保存起来供以后使用
2. 

# 状态和状态转移方程
如何拆分问题,是动态规划的核心。拆分问题,靠的就是状态的定义和状态转移方程的定义。


# 应用
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

1. 拆分问题
想求数列第n项的值,先求n-1和n-2项的值,问题本身已经被拆分了。
2. 定义状态
定义F(n)为数列第n项的值
3. 定义状态转移方程
F(n) = F(n-1) + F(n-2)
F(0) = 0
F(1) = 1

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




### 最长递增子序列-dp
Longest increasing subsequence  

1. 拆分问题
求的是一个数列An的最长递增子序列(LIS),那么把子数列(A1-A2)、(A1-A3)...(A1-An)的LIS都求出来后取最大值,就是数列An的LIS
2. 定义状态
定义以Ai结尾的子数列(A1-Ai)的LIS是L(i)
3. 定义状态转移方程
L(i) = max({L(j) + 1}),其中j的取值范围是1到(i-1),并且Ai>Aj。
可以这么想:欲求以Ai结尾的LIS,需要遍历i之前的所有位置j,找到所有的Aj<Ai的位置;计算这些满足条件的i对应的LIS,然后取出最大值,将最大值加1就得到了以Ai结尾的LIS,即L(i)。(为什么要加1,因为寻找的都是比Ai小的Aj;所以再加上Ai自己,就是以Ai结尾的数列的LIS)。  
边界状态L(1)=1。  

由于该算法在每个位置都要往前遍历所有位置,所有它的时间复杂度是O(n<sup>2</sup>)
```
def lis_dp(sequence):
    length = len(sequence)
    # 保存以原序列sequence中每个元素结尾的子数列的lis,默认值为1
    sub_sequence_lis = [1] * length

    for i in xrange(0, length):
        max_len = 0

        # 遍历Ai前面的每一个元素Aj
        for j in xrange(0, i):
            # 需要满足Aj<Ai并且LSI最大的子序列。找到LSI最大的子序列之后,加上1就是Ai的LIS了。
            if sequence[i] > sequence[j] and max_len < sub_sequence_lis[j]:
                max_len = sub_sequence_lis[j]
        sub_sequence_lis[i] = max_len + 1

    # sub_sequence_lis的最大值就是原sequence的lsi
    return max(sub_sequence_lis)
```



### 最长递增子序列-nlogn方法
[参考](https://www.felix021.com/blog/read.php?1587)

```
 def lis_nlogn(sequence):
    lis = 1
    length = len(sequence)
    lis_location = [0] * (length + 1)
    lis_location[1] = sequence[0]

    for i in xrange(1, length):
        # 先找到插入的位置
        position = get_position(lis_location, 1, lis, sequence[i])
        # 在找到的位置插入元素
        lis_location[position] = sequence[i]

        # 如果是在
        if lis < position:
            lis = position
    return lis


# 在lis_location有序序列上,找到第一个大于等于key的位置;如果都小于key,返回end+1
def get_position(lis_location, start, end, key):
    if lis_location[end] <= key:
        return end + 1
    while start < end:
        mid = start + (end - start) / 2
        if lis_location[mid] <= key:
            start = mid + 1
        else:
            end = mid
    return start
```




### 背包问题

### Floyd-Warshall算法

### Viterbi算法
