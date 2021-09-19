# golang-1

> 这是一篇旧文，写于 2021 年 3 月 4 日。
> 最后更新于 2012 年 9 月 19 日。

## 安装

我使用 chocolatey 的包管理器安装：

```powershell
choco install golang -y
```

其中 -y 是自动确认。等待安装完成后，golang 就已经安装成功了。

安装完毕后，“C:\Program Files\Go\bin\”会被 choco 加入环境变量，不需要专门处理。重启命令行之后，go 就可以开始使用了。

输入`go version`，得到`go1.15.8 windows/amd64`或者类似的反馈，说明安装成功。

---

更新：

- 现在更加建议使用 winget 安装；
- 现在 go 的版本号达到了 go1.17。


## 设置包管理和代理

因为 go 是 google 的服务，在网络环境比较魔法的情况下下包体验不佳，需要设置代理。

如下：

```powershell
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

可以使用`go env`命令检查是否修改成功。这个命令也可以修改一些其它环境，此处不赘述。

---

更新：随着 go 的版本发展，现在这个“模块”功能已经是自动开启的了。
补充：为什么这个参数名字这么诡异呢，因为这个功能是从 go1.11 才开始加入的。

## 创建mod文件

使用`go mod init "domin/name"`来初始化包管理信息。
初始化成功后，会生成"go.mod"文件来存放需求信息。

## gin 和其它模块的安装

使用`go get -u github.com/gin-gonic/gin`。这一步需要GO111MODULE设置为“on”才能正常运行。
运行成功后，会在"go.mod"文件中写入需求，并且生成"go.sum"文件。此文件中每一行包含一组【包名称 包版本 包的哈希】。
注意：此过程需要管理员权限，否则报错类似如下：

```powershell
github.com/gin-gonic/gin@v1.6.3: verifying module: github.com/gin-gonic/gin@v1.6.3: open C:\Program Files\Go\pkg\sumdb\sum.golang.org\latest: Access is denied.
```

## 关于 mysql

记得安装驱动，安装后要使用`import _ "github.com/go-sql-driver/mysql"`来进行初始化。
格式是 import alias pack,alias 用于设置别名，类似于 py 的 import pack as alias.
下划线在 go 里被丢弃，所以这时候包里的定义 什么都获取不到，可以防止误用。

## 关于默认输出

不要用 defer 来把默认输出放在最前面，否则最后会给俩 json，实际应用接受到会报错

## 关于密码验证

首先显然不能直接存明文密码，要存密码的某种哈希。这个哈希最好加盐。

`bcrypt.GenerateFromPassword(password []byte, cost int)`函数是一种加盐的哈希实现，可以安全的存密码。

加盐之后，即使对于相同的 password，生成的哈希也不同。哈希结果大概长这样（里面包含了盐和哈希两部分）：

> $2a$10$Ejr3lUXe6tikizjuItZ.E.8gA2J8bfhtbwDKWBThO2HRNdbIlmV..

想要验证密码和哈希结果是否匹配，使用 `CompareHashAndPassword(hashedPassword, password []byte)` 来计算。如果匹配这个函数返回空指针，否则返回报错。

## 关于多文件运行

Goland 里不止一个文件的时候运行配置里种类要选包或者目录，不能选文件。命令行里`go build`的时候要把所有文件都写上。

## 关于 token

token 需要一个字符串进行签名。对 TOKEN 对象调用.SignedString()方法来签名。
