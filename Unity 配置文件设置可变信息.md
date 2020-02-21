# Unity 配置文件设置可变信息

在日常的开发中，总有一些配置信息是需要根据情况改变的，例如数据库的连接信息在开发和上线时必然是不同的。如果把这些信息硬编码到代码里，那么简单地修改配置信息就意味着要重新打包（Unity，我说的就是你）。之前一个项目就遇到了需要修改 ip 地址的问题，当时难的倒不是配置文件的读取和写入，而是在 Unity 程序打包之后从哪里读取配置文件的问题（当时写的绝对路径，没脸看了）。

<!-- more -->

## 配置文件格式的选择

目前有很多类型的配置文件，如 java 有 properties，JS 有 json，此外还包括 xml，ini 等。就普通人的可读性而言，我认为 ini 是这其中最优秀的，尽管他在配置的嵌套方面效果很差，但是在普通 Unity 项目中也很少会遇到这样的需求。

本文就以 ini 配置文件配置 MySQL 数据库连接信息为例，简单介绍一下如何在 Unity 中解析配置文件，并且在项目打包之后如何读取的问题。

## 基础环境设置

1. 我们沿用之前配置 Rider+Unity+MySQL8 环境所使用的 Unity 工程，其 Assets 文件夹下的目录结构是这样的。

    ```bash
    D:.
    │  Plugins.meta
    │  Scenes.meta
    │  Scripts.meta
    │
    ├─Plugins
    │  │  BouncyCastle.Crypto.dll
    │  │  BouncyCastle.Crypto.dll.meta
    │  │  Editor.meta
    │  │  Google.Protobuf.dll
    │  │  Google.Protobuf.dll.meta
    │  │  MySql.Data.dll
    │  │  MySql.Data.dll.meta
    │  │  Renci.SshNet.dll
    │  │  Renci.SshNet.dll.meta
    │  │
    │  └─Editor
    │      │  JetBrains.meta
    │      │
    │      └─JetBrains
    │              JetBrains.Rider.Unity.Editor.Plugin.Repacked.dll
    │              JetBrains.Rider.Unity.Editor.Plugin.Repacked.dll.meta
    │
    ├─Scenes
    │      Main.unity
    │      Main.unity.meta
    │
    └─Scripts
            MySQLConnector.cs
            MySQLConnector.cs.meta
    ```

2. ini 文件的读取解析我选择的是 Unity Asset Store 中的 [Advanced INI Parser](https://assetstore.unity.com/packages/tools/advanced-ini-parser-23706)，直接在 Asset Store 中下载 Advanced INI Parser 插件并导入到项目中，可以看到，这个插件轻量化到就只有一个 cs 文件，只需要把该文件放到 Assets/Scripts 文件夹中就可以了。
3. 为了简单行事，我就直接在 MySQLConnector.cs 这个脚本中写配置文件的读取解析以及在 Unity 界面上展示查询结果的代码了。

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

        private INIParser _iniParser;
        private string _singleInfo;

        private void Start()
        {
            _iniParser = new INIParser();
            var configPath = Application.dataPath+"/Configs/MySQLConfig.ini";
            _iniParser.Open(configPath);

            host = _iniParser.ReadValue("MySQL","host","undefined");
            username = _iniParser.ReadValue("MySQL","username","undefined");
            password = _iniParser.ReadValue("MySQL","password","undefined");
            schema = _iniParser.ReadValue("MySQL","schema","undefined");

            ConnectMysql(host, username, password, schema);
        }

        private void OnGUI()
        {
            GUI.Label(new Rect(100,100,200,100),_singleInfo );
            if (!GUILayout.Button(("查询"))) return;
            _singleInfo = QueryInfo("select * from `connect-unity` where name='taujiong'");
        }

        private string QueryInfo(string queryStr)
        {
            string result = null;
            _mySqlCommand = _connection.CreateCommand();
            _mySqlCommand.CommandText=queryStr;
            var dataReader = _mySqlCommand.ExecuteReader();
            while (dataReader.Read())
            {
                result = $"name={dataReader.GetString(0)},age={dataReader.GetString(1)},score={dataReader.GetString(2)}";
            }

            return result;
        }

        private void ConnectMysql(string server, string user, string pwd, string database)
        {
            var connStr = $"Server = {server}; Database = {database}; User ID = {user}; Password = {pwd};";
            _connection = new MySqlConnection(connStr);
            try
            {
                print("connecting sql");
                _connection.Open();
                print(_connection.ServerVersion);
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
                throw;
            }
        }
    }
    ```

    相较于之前的代码，改动主要在以下几个方面：

    1. 在 Start 方法中引入了 INIParser 对象并使用他来读取解析 MySQLConfig.ini 文件，获取需要的值赋值给数据库连接参数。
    2. 添加了 OnGUI 方法来在界面上显示按钮以及查询结果。
    3. 将 QueryInfo 方法设置为返回单条查询结果。

4. 全部工作完成之后，Assets 目录下的结构变成这样。主要变动是增加了 /Configs/MySQLConfig.ini 以及 INIParser.cs。

    ```bash
    D:.
    │  Configs.meta
    │  Plugins.meta
    │  Scenes.meta
    │  Scripts.meta
    │
    ├─Configs
    │      MySQLConfig.ini
    │      MySQLConfig.ini.meta
    │
    ├─Plugins
    │  │  BouncyCastle.Crypto.dll
    │  │  BouncyCastle.Crypto.dll.meta
    │  │  Editor.meta
    │  │  Google.Protobuf.dll
    │  │  Google.Protobuf.dll.meta
    │  │  MySql.Data.dll
    │  │  MySql.Data.dll.meta
    │  │  Renci.SshNet.dll
    │  │  Renci.SshNet.dll.meta
    │  │
    │  └─Editor
    │      │  JetBrains.meta
    │      │
    │      └─JetBrains
    │              JetBrains.Rider.Unity.Editor.Plugin.Repacked.dll
    │              JetBrains.Rider.Unity.Editor.Plugin.Repacked.dll.meta
    │
    ├─Scenes
    │      Main.unity
    │      Main.unity.meta
    │
    └─Scripts
            INIParser.cs
            INIParser.cs.meta
            MySQLConnector.cs
            MySQLConnector.cs.meta
    ```

## 效果展示

1. 首先，在 Unity 软件中运行，能够正常显示查询结果。

    ![Unity 中查询结果](https://i.loli.net/2020/01/08/pXId4aw9hJsT3yG.png)
2. 之后我们将项目打包到 build 文件夹下，打包之后，其目录结构是这样的。

    ![打包后目录结构](https://i.loli.net/2020/01/08/cmKgVzq3l6LwhC2.png)
3. 此时我们直接运行程序是不能成功显示查询信息的，因为 Unity 打包之后并不能将 /Assets/Configs/MySQLConfig.ini 一起打包，而是需要我们自己在 Unity-Connect-MySql8_Data 文件夹下创建 /Configs/MySQLConfig.ini 文件，并写入自己需要的配置信息。~~之后，再次运行程序，结果就和预期的一样了~~。
4. 出现新坑，按照上述步骤完成之后，点击查询按钮没有反应。
   1. 经过 debug，确认了不是配置文件读取的问题，因为能够打印出来配置文件中的信息。
   2. 不是网上说的需要添加若干 I18N 库文件的问题，因为添加了还是没反应。
   3. 最终确认是打包时设置的问题，需要将 Api Compatibility Level 设置为 `.Net 4.x` 而不是 `.Net Standard 2.0`（仅针对 Unity2018.4.14f1，不同版本这里的选项可能不同）。
5. 具体操作为 File -> Build Settings -> Player Settings -> Other Settings -> Configuration

    ![api 配置](https://i.loli.net/2020/01/08/qWz1k93BEb8avYm.png)
6. 现在，重新打包并将文件放置在指定的位置就能产生预期的结果了。

    ![打包后连接正常](https://i.loli.net/2020/01/08/1OslwcSnG9rpB3P.png)

## 总结

这篇文章其实还是之前欠下来的技术债。看着写了很多，但其实操作很少。关键点是以下几条：

1. 使用 Advanced INI Parser 读取和解析 ini 配置文件 -> 将 INIParser.cs 脚本文件放置到 Assets/Scripts 文件夹下。
2. 读取配置文件时使用 Application.dataPath+ ${文件相对于 Assets 文件夹的相对路径}（关于为什么是这样可以参考 [unity 的特殊目录和脚本编译顺序](https://www.jianshu.com/p/7494cd1594d3)）。
3. 项目打包之后，在 ${项目名称}_Data 文件夹下对应放置配置文件，然后运行程序即可。
4. 特别地，如果项目中需要用到数据库，要注意打包时 Api Compatibility Level 的设置。
