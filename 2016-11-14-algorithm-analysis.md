---
title: 算法分析
date: 2016-11-14 15:26:14
categories: algorithm
tags: algorithm
---

算法的时间复杂度分析

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

## Demo
### Ex.1


```
int foo(int n) {
    if (n <= 1) {
        return 1；
    }
    return n * foo(n - 1);
}
```
基本语句是if语句，该语句执行了n次，所以时间复杂度为O(n)
### Ex.2
```
void recursive(int n, int m, int o) {
    if (n <= 0) {
        printf("%d, %d\n", m, o);
    } else {
        recursive(n - 1, m + 1, o);
        recursive(n - 1, m, o + 1);
    }
}
```
算法的执行时间T(n, m, o) = T(n-1, m+1, o) + T(n-1, m, o+1)，执行时间的上界和m、o的关系不大，可以简化为T(n)=2T(n-1)，可以知道，T(n)=2T(n-1) = 2<sup>2</sup>T(n-2) = ... = 2<sup>n</sup>T(1) = 2<sup>n+1</sup>T(0)。所以，算法的时间复杂度为O(2<sup>n</sup>)。
### Ex.3
```
# 其中m > 1，e > 0
x = m;
y = 1;
while (x - y > e) {
    x = (x + y) / 2;
    y = m / x;
}
print(x);
```
由于m>1，所以x = (x + y) / 2语句会让x的值变小；m不变的情况下，y = m / x语句会让y的值变大，x-y的值会越来越小。假设x >> y，y = 1，那么x = (x + y) / 2语句相当于每次让x减半，所以循环执行的次数不会超过logm；y = m / x语句会让y的值变大的情况下，循环执行的次数更不会超过logm了。所以，算法的时间复杂度是logm。
### Ex.4

```
假设某算法的计算时间可用递推关系式T(n) = 2T(n/2) + n，T(1) = 1表示，则该算法的时间复杂度为？
```

T(n) = 2T(n/2) + n = 2(2T(n/4) + n/2) + n = ... = 2<sup>m</sup>T(n/2<sup>m</sup>) + m * n  
递归到T(1)时，n = 2<sup>m</sup>，即m = log(n)  
所以T(n) = nT(1) + nlog(n)，该算法的时间复杂度是nlog(n)
### Ex.5

```
假设某算法的计算时间可用递推关系式T(n) = 25T(n/5) + n*n，T(1) = 1表示，则该算法的时间复杂度为？
```

T(n) = 25T(n/5) + n<sup>2</sup> = 25(25T(n/5/5) + (n/5)<sup>2</sup>) + n<sup>2</sup> = 5<sup>4</sup>T(n/5<sup>2</sup>) + 2n<sup>2</sup> = ... = 5<sup>2m</sup>(n/5<sup>m</sup>) + m*n<sup>2</sup>   
递归到T(1)时，K<sub>m</sub>/5<sup>m+1</sup> = 1，由于K<sbu>m</sub>=n/5<sup>m</sup>，所以n = 5<sup>m</sup> ,即m = log<sub>5</sub>(n)   
所以T(n) = n<sup>2</sup>T(1) + n<sup>2</sup>log<sub>5</sub>(n)，该算法的时间复杂度是n<sup>2</sup>log(n)
### Ex.6

```
假设某算法的计算时间可用递推关系式T(n) = 1 + T(n/2) + n*n，T(1) = 1表示，则该算法的时间复杂度为？
```  

T(n) = 1 + T(n/2) + n<sup>2</sup> = 1 + (1 + T(n/2<sup>2</sup>) + (n/2)<sup>2</sup>) + n<sup>2</sup> = ... = m + T(n/2<sup>m</sup>) + n<sup>2</sup>(1+1/2<sup>2</sup>+...+1/2<sup>m</sup>)  
显而易见，该算法的时间复杂度是n<sup>2</sup>

