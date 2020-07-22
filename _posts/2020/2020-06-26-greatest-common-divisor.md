---
layout: post
title: 小灰 - 5.4 求最大公约数
category: algorithm
tags: [algorithm]
excerpt: Greatest Common Divisor
---

## 问题描述  

如何求两个整数最大公约数？  


举个例子：  

``` java
a = 20
b = 15
Ouput = 5
```


## 解法1 - 暴力法  

首先，获取`a`与`b`的较小值`small`。  

然后，从`i = small / 2`依次遍历，直到`a`、`b`与`i`取余均为`0`，或者`i = 1`终止。  


来看下实现：  


``` java
private static int gcdV1(int a, int b) {

    int big   = Math.max(a, b);
    int small = Math.min(a, b);

    for (int i = small / 2; i > 1; i--){
        if (big % i == 0 && small % i == 0){
            return i;
        }
    }
    return 1;
}
```

该解法有什么缺点？  

如果`a = 1000`, `b = 1001`，需要循环`1000 / 2 - 1 = 499`次，而且结果还是`1`。  


## 解法2 - 辗转取余法  

首先，分别获取较大值`big`、较小值`small`。  

然后，判断`big % small == 0`，如果等于`0`，则`small`就是答案，  

否则继续递归： `gcd(big % small, small)`  

来看下实现：  


``` java
private static int gcdV2(int a, int b) {

    int big   = Math.max(a, b);
    int small = Math.min(a, b);

    if (big % small == 0){
        return small;
    }

    return gcdV2(big % small, small);
}
```

### Update at 2020_0722  

循环法：  

假设`a`为较大值，`b`为较小值。  

`while`循环结束的条件为`b != 0`，  

在循环中，先计算`a`与`b`取余的结果，然后`a = b; b = t`  

因为`b`肯定比`t`大。  

``` java
a = Math.max(a, b);
b = Math.min(a, b);
while (b != 0){
    int t = a % b;
    a = b;
    b = t;
}
```



这种解法有什么问题？  

对于两个大值取余的效率太低。  



## 解法3 - 更相减损法  


这种解法解决了两个大数取余的效率问题。  

首先，也是分别获取较大值`big`，较小值`small`。  

然后，判断`big == small`，若相等，则找到了答案。  

否则，继续递归遍历： `gcd(big - small, small);`  

来看下实现：  


``` java
private static int gcdV3(int a, int b) {
    if (a == b) return a;

    int big   = Math.max(a, b);
    int small = Math.min(a, b);

    return gcdV3(big - small, small);
}
```

但是，这种解法的有什么缺点？  

如果两个数相差很大，则需要大量递归运算。  



### Update at 2020_0722  

重新梳理下思路：  

将`a`设为较大值，`b`为较小值。  

然后，`while`循环结束的条件就是两者相等，  

在每次循环中，先计算两者差值`t`，然后`a = Max(b, t), b = Min(b,t)`  

``` java
a = Math.max(a, b);
b = Math.min(a, b);
while (a != b){
    int t = a - b;
    a = Math.max(t, b);
    b = Math.min(t, b);
}
```


## 解法4 - 更相减损法 & 移位  

最后，来看一种最合适的解法。  

首先，判断`a`和`b`的`奇偶性`，分为四种情况：  

如果`a`和`b`都是偶数，那么它们肯定有一个公约数`2`，所以：  

``` java
return gcdV4(a >>> 1, b >>> 1) << 1;
```

如果，`a`和`b`一个奇数，另一个是偶数，则偶数除以`2`，继续递归。  

``` java
else if ((a & 1) == 0 && (b & 1) == 1){
    return gcdV4(a >>> 1, b);
} else if ((a & 1) == 1 && (b & 1) == 0){
    return gcdV4(a, b >>> 1);
} 
```

如果`a`和`b`都是奇数，此时就要使用`更相减损法`：  

``` java
else {
    int big   = Math.max(a, b);
    int small = Math.min(a, b);
    return gcdV4(big - small, small);
}
```

最后，再来考虑下递归函数的终止条件？  

当`a`与`b`相等：  


``` java
if (a == b) return a;
```

来看下完整实现：  


``` java
    private static int gcdV4(int a, int b) {

        if (a == b) return a;

        if ((a & 1) == 0 && (b & 1) == 0){
            return gcdV4(a >>> 1, b >>> 1) << 1;
        } else if ((a & 1) == 0 && (b & 1) == 1){
            return gcdV4(a >>> 1, b);
        } else if ((a & 1) == 1 && (b & 1) == 0){
            return gcdV4(a, b >>> 1);
        } else {
            int big   = Math.max(a, b);
            int small = Math.min(a, b);
            return gcdV4(big - small, small);
        }
    }
```

`Enjoy it ! `