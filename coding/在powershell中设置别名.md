#! https://zhuanlan.zhihu.com/p/406066200
# powershell alias

>  本文是一篇旧文，写于 2021 年 1 月 29 日。
> 最后更新于 2021 年 9 月 2 日。

地球人（？）都知道，要在bash里做的设置可以写在`~/.bashrc`里。

不过，在windows下的powershell平台里，可就没那么方便了。不得不说，windows下命令行的使用体验虽然在近几年以来有了一些进步，但是还是和linux差得远。

对于powershell/pwsh，类似的文件目录为`~/Documents/WindowsPowerShell/Microsoft.PowerShell_profile.ps1`可以调用 $profile 来获取这个地址。

```powershell
echo $profile
```
然后写入想要的任何预设启动脚本，例如：

```powershell
set-alias np notepad++.exe;
set-alias ll ls;
```

保存，重新打开powershell即可。要补充一点就是，powershell在设计的时候更加注重作为脚本的应用，因此pwsh在运行的时候可以改变不同的安全等级。

如果安全等级太高，启动的时候可能报错。使用`Get-ExecutionPolicy`来查看安全等级，默认为“restrict”则代表禁止一切脚本（ps1文件的执行）。

`Set-ExecutionPolicy remotesigned`，可以将其设置为允许运行所有本地脚本，远程脚本需要数字签名（确保内容没有被篡改），这样就能愉快的设置别名（和其它预处理脚本）了！

最后分享一下我现在用的配置：

```powershell
# system
set-alias ll ls;
set-alias which get-command

# third-party
set-alias np notepad++.exe;


# functions

# simple working directory
# 用pwd返回的是一个对象
function swd{"$pwd"}

# winget auto complete
# https://github.com/microsoft/winget-cli/blob/master/doc/Completion.md
Register-ArgumentCompleter -Native -CommandName winget -ScriptBlock {
    param($wordToComplete, $commandAst, $cursorPosition)
        [Console]::InputEncoding = [Console]::OutputEncoding = $OutputEncoding = [System.Text.Utf8Encoding]::new()
        $Local:word = $wordToComplete.Replace('"', '""')
        $Local:ast = $commandAst.ToString().Replace('"', '""')
        winget complete --word="$Local:word" --commandline "$Local:ast" --position $cursorPosition | ForEach-Object {
            [System.Management.Automation.CompletionResult]::new($_, $_, 'ParameterValue', $_)
        }
}
```
