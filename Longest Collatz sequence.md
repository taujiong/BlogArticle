# Longest Collatz sequence

[Project Euler](https://projecteuler.net) 第 14 题

<!-- more -->

## Question

The following iterative sequence is defined for the set of positive integers:

n → n/2 (n is even)
n → 3n + 1 (n is odd)

Using the rule above and starting with 13, we generate the following sequence:

13 → 40 → 20 → 10 → 5 → 16 → 8 → 4 → 2 → 1

It can be seen that this sequence (starting at 13 and finishing at 1) contains 10 terms. Although it has not been proved yet (Collatz Problem), it is thought that all starting numbers finish at 1.

Which starting number, under one million, produces the longest chain?

NOTE: Once the chain starts the terms are allowed to go above one million.

## Analysis

1. 为了简化计算，应该将已经计算过的值所需的步数保存在一个字典里面，这样越到后面计算起来越方便。
2. 初始值可以设在 500000，因为对于任意小于 500000 的数 $n$ ，必然有 $2n < 1000000 $的长度大于$ n$。
3. 递归的使用
4. [官方思路](https://projecteuler.net/overview=014)

## Program

```python
box = {1: 1}
result = 0
max = 0


def Collatz(num):
    if num in box.keys():
        return box[num]
    elif num % 2 == 0:
        step = Collatz(num / 2) + 1
    else:
        step = Collatz((3 * num + 1) / 2) + 2
    box[num] = step
    return step


for i in range(500000, 899999):
    step = Collatz(i)
    if step > max:
        result = i
        max = step

print(result)
```
