# Unity-Android 调试指南

最近实验室在基于 unity 平台开发一款应用，需要有 Windows 端和 Android 端两个版本。之前的常态是集中开发 Windows 端，在 Windows 上调试通过后直接 build 一个 Android 版本的 apk 文件，下载到办公室仅有的几步安卓手机上测试运行。这存在着以下几个问题：

* 开发效率低。每次 build 出来的包都有四五百兆大小，往手机端的传输就需要很久。
* 系统版本单一。众所周知，年轻人的手机通常都运行最新的 Android 系统，这样就导致在测试时只能测试 Android9.0 的系统，可能出现潜在 bug 找不到的情况。
* 无法调试，这也是本次文章的重点。

我们的需求是，点击按钮，调用系统默认的应用打开本地 pdf 文件。在 Windows 端一切正常，而打包到 Android 端之后，怎么点击按钮都没有反应。由于打开文件的函数中存在多个步骤，Android 端无法调试导致了不能准确定位问题所在。

<!-- more -->

## 2 解决方案

### 2.1 前期准备

1. unity，软件版本较新即可，当时使用的版本是 2018.2.9f1；教程使用的版本是 2019.1.11f1。
2. Visual Studio 2019 Community，需要下载相应版本的 vstu（visual studio tools for unity），以及安卓模拟器。

### 2.2 核心思路

1. 使用软件提供的安卓模拟器连接 unity 编辑器，实现 unity 打包之后自动部署到安卓模拟器。
2. 使用 Visual Studio 连接安卓模拟器进程，实现断点调试。

### 2.3 详细过程

#### 2.3.1 配置 visual studio

1. 点击 vs2019 软件界面状态栏的“工具——获取工具和功能”，选择使用“unity 的游戏开发”以及“使用.NET 的移动开发”。

    ![vs安装组件](https://i.loli.net/2020/01/08/DixAQgrcaZ69yVp.png)
2. 安装完组件之后，再次打开 vs，点击状态栏的“工具——Android——Android SDK 管理器”，根据自己的需求安装相应的 SDK。

    ![安装SDK](https://i.loli.net/2020/01/08/NB6oDHGv2C3ugVa.png)
3. 点击状态栏的“工具——Android——Android Device Manager”，新建一个对应 api 的安卓模拟器。  

    ![设置模拟器](https://i.loli.net/2020/01/08/ykSmlKMv3iBRCjL.png)
4. 初次使用时 VS 会要求你安装 Android Emulator 工具，同意即可。安装完成后，点击启动 Android Emulator 工具，成功后显示如图。  

    ![安卓模拟器启动](https://i.loli.net/2020/01/08/PCi3nMGN4U7yaJ8.png)

#### 2.3.2 连接模拟器与 unity

1. 在 unity 中打开“Edit——Preference——External Tools”，配置 Android 环境的路径。  
    PS：2018 版本需要自行上网下载安装，2019 版本在使用 unity hub 安装时可以直接配置上。
2. 打开“File——Build Settings”，切换到 Android 平台，Run Device 选项选择出现的的 Google Android SDK build (emulator 5554)。完成此步操作后，每次需要调试时直接点击“File——Build and Run（Ctrl+B）”就可以快速 build 并自动在安卓模拟器中运行了。  

    ![连接模拟器与Unity](https://i.loli.net/2020/01/08/U5Dyr36BM1OTswZ.png)
3. 更多 build app 的选项在窗口左下角的“Player Settings”选项中设置，其中包括设置 app 的目标系统版本以及系统的 cpu 架构。

#### 2.3.3 配置 vs 的 debug 功能

1. 同样点击 unity 软件中的“File——Build Settings”，选中下图中的三项。  

    ![debug设置](https://i.loli.net/2020/01/08/4HcSBUkZ8M7pRnf.png)
2. 按下 Ctrl+B 可以快速 build 并自动部署到安卓模拟器中运行。
3. 在 vs 软件中需要的代码行打上断点，并点击状态栏的“调试——附加 unity 调试程序”，正常情况下弹出的窗口中会显示出模拟器的选项，选中并确定即可进行调试了。  

    ![连接Unity实例调试](https://i.loli.net/2020/01/08/oivPa49YcWbK6IU.png)

### 2.4 其他

1. 使用 Android Studio 主要是替换 vs 提供的 SDK 以及模拟器功能，这方面的内容可以自行上网了解，注意 unity 调试还是需要在 vs 中进行。
2. 目前 unity 官方文档中还提到了 unity remote 的功能，好像是可以不用 build，只需要在 Android 端安装一个 unity remote 的应用就可以实时调试，本人尚未测试，有兴趣的可以自行了解。

## 3 总结

unity 官方建议的代码编辑器是 vs，而自 vs2017 以来，这款宇宙第一 IDE 已经实现了模块化的安装，大大降低了硬盘空间需求，所以如果仅仅是开发 unity-Android 程序的话，还是建议大家使用 vs，有什么需要的组件可以按需安装。
