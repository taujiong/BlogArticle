# Unity 使用 Rider 连接 MySQL8

最近在研究使用 jetbrains 家的 Rider IDE 开发 Unity 的工作流。碰巧之前项目中遇到了 Unity 连接数据库的问题，当时使用的是 MySQL5.x 的版本，搭建开发环境时磕磕绊绊，迫于时间原因没有停下来搞明白。现在终于把项目完成了，就想着好好把之前得过且过的问题给彻底解决掉。MySQL5.x 的问题有师兄在写文档，那么我就借着这个机会试试 Rider + Unity + MySQL8 的开发环境搭建。

<!-- more -->

## 基础环境简介

首先，怎样建立 Rider 和 Unity 之间的联系，使得 Unity 能够默认打开 Rider 的基础教程就不说了。

其次，我们已经本地服务器上搭建了一个 MySQL8 的服务器，其基本连接信息如下：

host:127.0.0.1  
port:3306  
username:test  
password:test2020  
schema:test  

最后，test 数据库下使用以下命令建立了一张表。

```sql
-- 使用Rider自带的数据库连接工具建的
CREATE TABLE `connect-unity` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(10) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `score` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=latin1
```

表中包含以下三条数据。

![data](https://i.loli.net/2020/01/08/u2lxhpQRO16WVGo.png)

至此，一个演示环境就搭建完成了。

## 在 Unity 项目中添加必要引用

这里需要强调的是，按照网上 MySQL5.x 的教程操作是会报错的。我按照下面的步骤则一切正常。

1. 使用 Rider 的 NuGet 包管理器搜索 MySQL，选择 MySQL.Data。第一次使用时我惊奇地发现这货竟然没有安装的地方！！！

   ![NuGet无安装按钮](https://i.loli.net/2020/01/08/nH8s35rIud7qoZL.png)

   研究之后才知道，对于一个新的 Unity 项目，你至少需要添加一个脚本文件，Rider 才能开始加载工具集，然后你才能将 MySQL.Data 包安装在相应的工具集中。

   ![NuGet有安装按钮](https://i.loli.net/2020/01/08/iBHaW8qRK6JMhPs.png)

   此时，我们将 MySQL.Data 安装到 Assembly-CSharp 中。
2. 安装完成之后，你将在 Packages 文件夹下找到四个包。

    ![MySQL.Data安装完成](https://i.loli.net/2020/01/08/lb6cSrfYPsCxB8N.png)

    我们分别把这些包中的 dll 文件复制到 Plugins 文件夹下。  

    ![复制dll文件](https://i.loli.net/2020/01/08/IjeoJcpS7d8D421.png)

## 编写代码

好了，现在你可以在脚本中编写连接数据库的代码了。与 MySQL5.x 不同的是，你不再需要 `using System.Data` 了。下面是我连接数据库并查询的相关代码。

```cs
using System;
using UnityEngine;
using MySql.Data.MySqlClient;

public class MySQLConnector : MonoBehaviour
{
    private static string host;
    private static string username;
    private static string password;
    private string schema;

    private MySqlConnection _connection;
    private MySqlCommand _mySqlCommand;

    private void Start()
    {
        host = "127.0.0.1";
        username = "test";
        password = "test2020";
        schema = "test";

        ConnectMysql(host, username, password, schema);
        QueryInfo("select * from `connect-unity`");
    }

    private void QueryInfo(string queryStr)
    {
        _mySqlCommand = _connection.CreateCommand();
        _mySqlCommand.CommandText=queryStr;
        var dataReader = _mySqlCommand.ExecuteReader();
        while (dataReader.Read())
        {
            if (dataReader.HasRows)
            {
                print($"name={dataReader.GetString(0)},age={dataReader.GetString(1)},score={dataReader.GetString(2)}");
            }
        }
    }

    private void ConnectMysql(string server, string user, string pwd, string database)
    {
        var connStr = $"Server = {server}; Database = {database}; User ID = {user}; Password = {pwd};"; //默认端口可以不写
        _connection = new MySqlConnection(connStr);
        try
        {
            print("connecting sql");
            _connection.Open();
            print(_connection.ServerVersion); //连接成功之后可以打印MySQL服务器版本号
        }
        catch (Exception e)
        {
            Console.WriteLine(e);
            throw;
        }
    }
}
```

最终效果如下图所示。

![查询结果打印](https://i.loli.net/2020/01/08/lxtT5s3KU4mNLjz.png)

## 总结

其实整个例子中没有任何一点复杂的操作，但是由于 MySQL8 比较新，Rider 又是第一次接触，所以中间还是遇到了一些坑的。总体而言，Rider+Unity+MySQL8 的工作流还是很让人愉悦的。

* 相较于 MySQL5.x，MySQL8 在 Unity 或者说 C#中的集成容易多了。你不再需要去 Unity 的安装目录下复制 System.Data，System.Drawing 等 dll 文件了。
* 无论是 VS 还是 Rider，NuGet 包管理器的集成极大地简化了开发流程。
* Rider 相较于 VS 确实是开发者的福音，有种瞌睡了自动给你送枕头的感觉。他的自动完成，性能提示，代码优化提示等让我只需要专注代码的逻辑，其他的细节 Rider 这个保姆会给我提供一条龙服务的。
