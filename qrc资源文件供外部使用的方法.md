# qrc资源文件供外部使用的方法

从入学之初就跟着师兄一起做一个 PyQt5 的项目，学到了很多东西。自从上次项目打包遇到资源文件处理难题之后，就开始慢慢在项目中引入 Qt 提供的 qrc 资源文件方案。

最近在系统地学 PySide2（Qt for python 的官方绑定），在其官方教程中有一个[Data Visualization Tool Tutorial](https://doc.qt.io/qtforpython/tutorials/datavisualize/index.html)的数据可视化教程。在该教程中，用到了 pandas 库来读取一个本地 CSV 文件。教程本身提供的是相对路径读取文件方法，但本人想使用 qrc 资源文件的方式来引入该 CSV 文件，以培养习惯。不曾想，遇到了 pandas 无法读取 CSV 文件的错误，在解决过程中对 qrc 资源文件的运作方式有了更深的理解。

<!--more-->

## 问题重现

首先我的项目目录结构为

```bash
D:.
└─DataVisualization
    │  main.py
    │  
    ├─layouts
    │      main.ui
    │
    └─resources
        │  all_day.csv
        │  data.qrc
        │  data_rc.py
```

data.qrc 代码为

```xml
<RCC>
  <qresource prefix="data">
    <file>all_day.csv</file>
  </qresource>
</RCC>
```

在 main.py 中，按照正常的相对路径写法，能够正确打印 CSV 文件内容，而使用 qrc 资源文件方法则不可行，具体代码为

```python
import pandas as pd
from PySide2.QtCore import QFile, QIODevice

import DataVisualization.resources.data_rc

if __name__ == "__main__":
    data = pd.read_csv(":/data/all_day.csv")
    # data = pd.read_csv(r"DataVisualization\resources\all_day.csv")
    print(data)
```

运行此 main.py 报错 `FileNotFoundError` ，而使用注释内的语句则能够正常显示 CSV 文件内容。

```bash
FileNotFoundError: [Errno 2] File b':/data/all_day.csv' does not exist: b':/data/all_day.csv'
```

## 解决方案

经过长时间的谷歌，终于从[stack overflow](https://stackoverflow.com/questions/52950601/create-a-pandas-dataframe-from-a-qrc-resource-file)找到了解决方案及原因。

总的来说，就是因为作为 Qt 内部使用的资源管理方案，只有 Qt 本身知道如何正确地从该资源文件中得到所需文件正确的路径并读取，而外界库甚至 python 自身也不能得到文件的具体路径。

既然外界读不到信息是因为找不到文件的具体路径，那么解决方案也就呼之欲出了：跳过路径，直接让 Qt 程序告诉第三方库所需文件的内容。

```python
import io
import pandas as pd
from PySide2.QtCore import QFile, QIODevice

import DataVisualization.resources.data_rc

if __name__ == "__main__":
    file = QFile(":/data/all_day.csv")
    if file.open(QIODevice.ReadOnly):
        f = io.BytesIO(file.readAll().data())
        data = pd.read_csv(f)
        print(data)
```

值得注意的是，这里 pandas.read_csv()通过此方案可行的原因是，read_csv()支持传入的参数既可以是文件路径也可以是 buffer。对于其他某些接口可能必须要文件路径的则此路不通。

## 深入思考

### qrc 的原理

直接上结论：qrc 文件本身类似于一个 XML 格式的文本，其记录了所包含的每一个文件的路径。关键在于，使用 rcc/pyrcc5/pyside2-rcc 对该文件进行编译时，程序遍历 qrc 文件里面的每一个文件路径，将该文件的二进制内容写入到相应的输出文件里。

例如，本文使用的 all_day.csv 文件是一个纯文本文件，其文件大小为 56.4KB；包含该文件路径信息的 data.qrc 文件大小为 85 字节；通过 pyside2-rcc.exe 编译生成的 data_rc.py 文件大小为 70.7KB。此外，当我编译完成了 data_rc.py 之后，即使我删除 all_day.csv 文件，程序照样能够正常运行并读取 all_day.csv 里面的数据。除了 pyside2-rcc.exe 将 all_day.csv 的所有信息统统写进了编译生成的 data_rc.py 这种解释外，我实在想不到为什么一个 py 文件能够达到 70.7KB。

有了这样的结论，也就很好解释 qrc 文件的一些相关特性了，例如：

1. 目标程序里所有内嵌的资源文件都是只读的，在程序运行时不能修改资源里的文件，只能读取使用。
2. 当你修改了 qrc 资源文件包含的任意文件之后，你必须重复**添加-编译**的操作才能使更改生效。
3. 除了 Qt 之外的程序想要直接从编译后的文件里面获取信息几乎是不可能的，必须要借助 Qt 内部接口读取资源文件信息。

### qrc 使用的其他注意事项

1. 为了管理方便，需要编译进 qrc 资源文件的那些文件最好放在 qrc 的所在文件夹或者其子文件夹内。
2. 在不加前缀（prefix）的前提下，在程序中调用同级资源文件可以使用 `(":file_name")` 如 `file = QFile(":/all_day.csv")` ；调用子目录资源文件可以用 `(":path/to/file")` 如 `file = QFile(":/resources/all_day.csv")` 。  
3. 在加前缀的情况下，在程序中调用同级资源文件可以使用 `(":/prefix/file_name")` 如 `file = QFile(":/data/all_day.csv")` ；调用子目录资源文件可以用 `(":/prefix/path/to/file")` 如 `file = QFile(":/data/resources/all_day.csv")` 。
4. 对于文件大小超过 4M 的文件，不建议直接编译，而是使用选项 `-binary` ，具体文档看[这里](https://doc.qt.io/qt-5/resources.html#external-binary-resources)。另外，在 Qt 的 python 绑定中不支持 `-binary` 。

## 总结

作为一个 PySide2 的入门者，我还有很多东西要学习，包括 qrc 也还有很多方面是一知半解的。如果对于 qrc 有什么好的见解，或者文章中有什么错误，希望能够多多交流。
