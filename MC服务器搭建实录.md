# MC服务器搭建实录

最近自己搭了一个 mc 的服务器，看网上的教程里面也有不少没有看清楚的部分，于是根据自己的理解写就了这篇文章，以供参考。

本文假定，读者具有一定的命令行操作知识。

## 配置要求

为了能够进行可控的多人游戏，需要以下东西：

- 至少 1 核心 1 Ghz的处理器[^proc]
- 至少 1G 内存（指空余，下同）
- 200M 以上的磁盘空间，其中 150m 用于存放生成的世界。
- 带有公网 ip 的网络连接[^ip]

## 配置

确保系统已经安装了 JAVA：

```powershell
> java -version
java -version java version "1.8.0_261"
Java(TM) SE Runtime Environment (build 1.8.0_261-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.261-b12, mixed mode)
```

```shell
> java -version
openjdk version "11.0.9.1" 2020-11-04
OpenJDK Runtime Environment (build 11.0.9.1+1-Ubuntu-0ubuntu1.20.04)
OpenJDK 64-Bit Server VM (build 11.0.9.1+1-Ubuntu-0ubuntu1.20.04, mixed mode, sharing)
```

注意：本文不负责解决 java 版本不匹配问题，请自行解决。

下载官方提供的 jar 包：[服务器下载](https://www.minecraft.net/zh-hans/download/server)

此链接为最新版，如需求特定版本，请自行寻找。

## 第一次启动

将上面提到的 server.jar（你下下来的可能是其它名字，自行替换）放在想要的文件夹下，执行指令`java -jar server.jar nogui`启动。

游戏会在同目录下生成 eula.txt 文件，使用 vim 或其他编辑器，将文本内的“eula=false”改为“eula=true”来同意 eula，重新执行指令，等待一会，即可启动。

## 游玩

在相同版本的客户端打开游戏，在多人游戏中添加服务器，地址填写服务器地址，即可进入游戏。

如果使用云服务器，请确保开放了 25565 端口以保证能够正常连接。

在 server.properties 中“online-mode=xx”可以调整是否为离线模式，没有经过正版验证的玩家不能进入开启在线模式的服务器。注意：如果这个选项为 false，那么任何人都可以进入你的服务器，这个选项一般用于局域网。

如果进入游戏后发现不能破坏方块，可能是因为出生地保护。

在后台使用`/op [player]`可以给予指定玩家 op 权限，类似于单人模式中的作弊选项。

在后台使用/stop 可以关闭服务器并存档。

更多可以调整的参数可以参见 [这个链接](https://minecraft-zh.gamepedia.com/Server.properties)

## 其它

关于正版验证，第三方验证，插件登录和 mod 等内容，我懒得继续写了，以后再说吧

[^proc]:按占用比例，例如 1 核 2G 的处理器，要保证不开服的时候占用不超过 50%。
[^ip]:如果所有参与者都处于同一个局域网内部，这个部分可以跳过但是还是要确认自己的 ip 到底是什么。
