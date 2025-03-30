# PowerShell 环境变量

介绍如何使用 PowerShell 来查询、永久增删改系统或用户环境变量。

## 获取环境变量：

```powershell
GetEnvironmentVariable(env_var_name, env_var_target)
```

`env_var_name` 替换为你要查询的环境变量的名字；

`env_var_target` 从 `'Machine'` 和 `'User'` 中二选一 。

1. Machine 表示修改系统环境变量；
2. User 表示修改用户环境变量。

例如，查询系统环境变量 Path：

```powershell
[Environment]::GetEnvironmentVariable('Path', 'Machine')
```

> GetEnvironmentVariable 是静态类 System.Environment 的方法。
> 使用静态类需要使用方括号 [] 将类名括起来；
> 使用静态方法或属性需要在前面添加符号 ::。

也可以使用 `dir env:` 来查看所有环境变量（包括系统环境变量和用户环境变量）；

使用 `$env:path` 来查看环境变量 path 的值（包括系统环境变量 path 和用户环境变量 path）。

## 永久新建环境变量：

```powershell
SetEnvironmentVariable(env_var_name, env_var_value, env_var_target)
```

`env_var_name` 替换为你要新建的环境变量的名字；

`env_var_value` 替换为你要设置的环境变量的值；

`env_var_target` 同上，从 `'Machine'` 和 `'User'` 中二选一 。

> 注意：新建系统环境变量需要管理员权限！

例如，新建一个名为 `JAVA_HOME` 的系统环境变量，在 PowerShell 中执行以下命令：

```powershell
[Environment]::SetEnvironmentVariable('JAVA_HOME', 'C:\Program Files\Java\jdk-11.0.13' , 'Machine')
```

## 永久修改已有的环境变量的值：

如果环境变量已经存在，使用 `SetEnvironmentVariable` 就是修改它的值。

用法与新建环境变量完全相同。

例如，修改系统环境变量 Path，将上一个例子中 `JAVA_HOME` 路径下的 `bin` 目录添加到 Path：

```powershell
$path = [Environment]::GetEnvironmentVariable('Path', 'Machine')
$newpath = 'c:\systools;' + $path 
[Environment]::SetEnvironmentVariable('Path', $newpath, 'Machine')
```

不建议使用这种方法修改环境变量 Path，因为 Path 中引用的变量会被自动替换为变量的值，如 %windir% 会被替换为 C:\WINDOWS。

## 永久删除已有的环境变量的值：

只需要将环境变量的值设为 `$null`，就可以删除该环境变量。

例如，删除之前创建的系统环境变量 `JAVA_HOME` ：

```powershell
[Environment]::SetEnvironmentVariable('JAVA_HOME', $null, 'Machine')
```



注意：

1. 新建、修改、删除系统环境变量需要管理员权限，请使用管理员权限运行 PowerShell；
2. 新建、修改、删除环境变量在系统层面会立即生效，但在该 PowerShell 中并不，需要重启 PowerShell 才能在 PowerShell 中看到改变。



编辑于 2022-02-12 16:24