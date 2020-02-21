# Amicable numbers

[Project Euler](https://projecteuler.net) 第 21 题

<!-- more -->

## Question

Let d(n) be defined as the sum of proper divisors of n (numbers less than n which divide evenly into n).
If d(a) = b and d(b) = a, where a ≠ b, then a and b are an amicable pair and each of a and b are called amicable numbers.

For example, the proper divisors of 220 are 1, 2, 4, 5, 10, 11, 20, 22, 44, 55 and 110; therefore d(220) = 284. The proper divisors of 284 are 1, 2, 4, 71 and 142; so d(284) = 220.

Evaluate the sum of all the amicable numbers under 10000.

## Analysis

1. 因子求和函数应该怎样优化？
2. 字典来保存计算结果。
3. [官方思路](https://projecteuler.net/overview=021)

## Program

```python
from math import sqrt, ceil


def get_sum(num):
    sum = 1
    mid = sqrt(num)
    for factor in range(2, ceil(mid) + 1):
        if num % factor == 0:
            sum += factor
            if factor != mid:
                sum += num // factor

    return sum


max_size = 10000
result = 0
cal = {}

for i in range(1, max_size):
    if cal.__contains__(i):
        a = cal[i]
    else:
        a = get_sum(i)
        cal[i] = a

    if cal.__contains__(a):
        b = cal[a]
    else:
        b = get_sum(a)
        cal[a] = b

    if i == b and i != a:
        result += i

print(result)
```
