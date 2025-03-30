# 自定义powershell 环境

PowerShell 配置文件是在 PowerShell 启动时运行的脚本。 可以使用配置文件自定义环境。 方法：

- 添加别名、函数和变量
- 加载模块
- 创建 PowerShell 驱动器
- 运行任意命令
- 和更改首选项设置

将这些设置放在配置文件中可确保每次在系统上启动 PowerShell 时，它们都可用。

 备注

若要在 Windows 中运行脚本，至少需要将 PowerShell 执行策略设置为 `RemoteSigned`。 执行策略不适用于 macOS 和 Linux。 有关详细信息，请参阅 [about_Execution_Policy](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_execution_policies)。


## $PROFILE 变量

`$PROFILE` 自动变量存储当前会话中可用的 PowerShell 配置文件的路径。

有四种可能的配置文件可用于支持不同的用户范围和不同的 PowerShell 主机。 对于每个配置文件脚本，其完全限定的路径存储在以下成员属性 `$PROFILE`中。

- **AllUsersAllHosts**
- **AllUsersCurrentHost**
- **CurrentUserAllHosts**
- **CurrentUserCurrentHost**

可以创建为所有用户或只为一名用户（即 **CurrentUser**）运行的配置文件脚本。 **CurrentUser** 配置文件存储在用户的主目录中。

还有为所有 PowerShell 主机或特定主机运行的配置文件。 每个 PowerShell 主机的配置文件脚本在该主机上都具有唯一名称。 例如，Windows 上的标准 Console 主机或其他平台上的默认终端应用程序的文件名为 `Microsoft.PowerShell_profile.ps1`。 对于 Visual Studio Code (VS Code)，文件名为 `Microsoft.VSCode_profile.ps1`。

有关详细信息，请参阅 [about_Profiles](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_profiles)。

默认情况下，引用 `$PROFILE` 变量将返回“当前用户，当前主机”配置文件的路径。 可以通过 `$PROFILE` 变量的属性访问其他配置文件路径。 例如：

PowerShell

```powershell
PS> $PROFILE
C:\Users\user1\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
PS> $PROFILE.AllUsersAllHosts
C:\Program Files\PowerShell\7\profile.ps1
```



## 如何创建个人配置文件

首次在系统上安装 PowerShell 时，配置文件脚本文件和它们所属的目录不存在。 以下命令创建“当前用户，当前主机”配置文件脚本文件（如果不存在）。

PowerShell

```powershell
if (!(Test-Path -Path $PROFILE)) {
  New-Item -ItemType File -Path $PROFILE -Force
}
```

`New-Item` cmdlet 的 Force 参数会创建必要的文件夹（如果不存在）。 创建脚本文件后，可以使用喜欢的编辑器自定义 shell 环境。



## 向配置文件添加自定义项

前面的文章介绍了如何使用 [Tab 自动补全](https://learn.microsoft.com/zh-cn/powershell/scripting/learn/shell/tab-completion?view=powershell-7.3)、[命令预测器](https://learn.microsoft.com/zh-cn/powershell/scripting/learn/shell/using-predictors?view=powershell-7.3)和[别名](https://learn.microsoft.com/zh-cn/powershell/scripting/learn/shell/using-aliases?view=powershell-7.3)。 这些文章演示了用于加载所需模块、创建自定义补全程序、定义键绑定和其他设置的命令。 这些是你希望在每个 PowerShell 交互式会话中可用的自定义类型。 配置文件脚本是这些设置所在的位置。

编辑配置文件脚本的最简单方法是在喜欢的代码编辑器中打开该文件。 例如，以下命令在 [VS Code](https://code.visualstudio.com/) 中打开配置文件。

PowerShell

```powershell
code $PROFILE
```

还可使用 Windows 上的 `notepad.exe`、Linux 上的 `vi` 或任何其他文本编辑器。