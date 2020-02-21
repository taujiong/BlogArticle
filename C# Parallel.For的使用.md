# C# Parallel.For的使用

实验室接了一个新的项目，需要在一块 3d 地形上基于速度绘上颜色，而且要动态改变。由于地形比较大，mesh 网格的顶点一共有一千三百多万个，即使在逻辑上优化之后也达到了一百多万个。使用传统的 for 循环为每个顶点赋颜色值，效率太低，最终的 fps 只能达到 8 左右，完全达不到项目要求。在老师的指导下，开始考虑多线程的方法提高效率。

<!-- more -->

## Parallel.For

在查阅了大量网络资源之后，确定了最简单易行的方法是 C# 本身提供的 Parallel 并行计算方法。

关于 Parallel.For 的语法就不细说了，直接通过下面的代码见识一下并行的威力吧。

```cs
using System.Diagnostics;
using System.Threading.Tasks;
using UnityEngine;

public class testParallel : MonoBehaviour
{
    Stopwatch stopWatch = new Stopwatch();
    void Start()
    {
        stopWatch.Start();
        for (int i = 0; i < 10000; i++)
        {
            for (int j = 0; j < 60000; j++)
            {
                int sum = 0;
                sum += i;
            }
        }
        stopWatch.Stop();
        print("NormalFor run" + stopWatch.ElapsedMilliseconds + "ms.");

        stopWatch.Reset();
        stopWatch.Start();
        Parallel.For(0, 10000, item =>
        {
            for (int j = 0; j < 60000; j++)
            {
                int sum = 0;
                sum += item;
            }
        });
        stopWatch.Stop();
        print("ParallelFor run" + stopWatch.ElapsedMilliseconds + "ms.");
    }
}
```

在 unity 中运行该代码，其结果如下图所示

![两种for循环的对比](https://i.loli.net/2020/01/08/YfAhZG5C8xbEKkD.png)

可以发现，Parallel.For 相较于传统的 for 循环运行时间加快了 1s。而在本次项目中，在使用了 Parallel.For 的情况下，fps 直接达到了 23，这是一个相当大的提升了。

## 深入思考

在体验到 Parallel.For 的巨大优势之后，我不禁思考：

1. 是不是所有的 Parallel.For 都比传统的 for 循环都快呢?
2. 今后我是不是可以把所有的 for 循环都替换为 Parallel.For 呢?

带着这样的问题，我继续搜索，也发现了使用 Parallel.For 的一些注意事项。

### Parallel.For 的快是有前提的

众所周知，在实现多线程时，为了防止多个线程同时处理同一个变量而导致变量处于 "薛定谔状态"，引入了 "锁" 的概念，即在每一时刻只有获得 "锁" 的线程才能操作目标变量。那么如果在 Parallel.For 中也需要操作一个全局变量，就意味着即使这是并行计算，大家也需要排队操作全局变量，此时 Parallel.For 可能远远不如传统的 for 循环来的快。

```cs
using System.Collections.Concurrent;
using System.Diagnostics;
using System.Threading.Tasks;
using UnityEngine;

public class testParallel : MonoBehaviour
{
    Stopwatch stopWatch = new Stopwatch();
    void Start()
    {
        var obj = new Object();
        long num = 0;
        ConcurrentBag<long> bag = new ConcurrentBag<long>();

        stopWatch.Start();
        for (int i = 0; i < 10000; i++)
        {
            for (int j = 0; j < 60000; j++)
            {
                num++;
            }
        }
        stopWatch.Stop();
        print("NormalFor run" + stopWatch.ElapsedMilliseconds + "ms.");

        stopWatch.Reset();
        stopWatch.Start();
        Parallel.For(0, 10000, item =>
        {
            for (int j = 0; j < 60000; j++)
            {
                lock (obj)
                {num++;}
            }
        });
        stopWatch.Stop();
        print("ParallelFor run" + stopWatch.ElapsedMilliseconds + "ms.");
    }
}
```

在 unity 中运行该代码，其结果如下图所示

![加了锁的Parallel.For与传统for的比较](https://i.loli.net/2020/01/08/9gRCnkubHrYd1j6.png)

可以发现，Parallel.For 的运行时间竟然是传统 for 循环运行时间的 100 倍！因此可以得出结论：**如果在循环中需要竞争资源，用到线程锁的话，Parallel.For 未必优于传统 for 循环**。

### Parallel.For 的循环是无序的

由于 Parallel.For 中的各个循环是同时进行的，所以每次循环体执行的时间会有些许差异，这就导致了循环的运行是无序的。

```cs
using System.Threading.Tasks;
using UnityEngine;

public class testParallel : MonoBehaviour
{
    void Start()
    {
        Parallel.For(0, 5, item =>
        {
            print(item);
        });
    }
}
```

在 unity 中运行该代码，其结果如下图所示

![循环的无序性](https://i.loli.net/2020/01/08/ExWqDC4yR8PtIY6.png)

因此可以得出结论：**如果循环的执行顺序需要严格控制的话，则不能使用 Parallel.For**。

## 总结

在计算机多核处理器普及的前提下，合理的使用多线程或者并行能够显著提高软件的运行效率。当然，这一切都是有前提的。滥用多线程往往会适得其反，不仅得不到理想的输出结果，甚至会干扰其他程序的正常运行。在明确了自己的需求的前提下，选择合适的方法才是最重要的。
