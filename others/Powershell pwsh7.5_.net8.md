TEST

在 PowerShell 7.5.0 中调用 .NET 8 的类方法，你可以按照以下步骤和示例进行操作：

### 1. 确保环境配置正确

首先要保证系统中已经安装了 .NET 8 运行时，并且 PowerShell 7.5.0 能够正常使用。

### 2. 引用命名空间和创建对象

如果要使用 .NET 8 中的类，通常需要先引用对应的命名空间，然后创建类的实例对象，最后调用对象的方法。

#### 示例 1：调用 `System.DateTime` 类的方法

`System.DateTime` 是 .NET 中用于处理日期和时间的类，以下是如何在 PowerShell 7.5.0 中调用它的方法：

```powershell
# 获取当前日期和时间
$currentDateTime = [System.DateTime]::Now
# 调用 ToString 方法将日期时间转换为字符串
$dateTimeString = $currentDateTime.ToString("yyyy-MM-dd HH:mm:ss")
Write-Output "当前日期和时间: $dateTimeString"
```

在这个示例中：

- `[System.DateTime]::Now` 用于获取当前的日期和时间，`::` 符号用于访问静态成员。
- `$currentDateTime.ToString("yyyy-MM-dd HH:mm:ss")` 调用了 `DateTime` 对象的 `ToString` 方法，将日期时间按照指定的格式转换为字符串。

#### 示例 2：调用自定义 .NET 类库的方法

假设你有一个使用 .NET 8 编写的自定义类库，其中包含一个简单的类和方法。

**步骤 1：创建 .NET 8 类库项目**
首先，使用以下命令创建一个 .NET 8 的类库项目：

```sh
dotnet new classlib -n MyNet8Library -f net8.0
```

然后，打开 `MyNet8Library.csproj` 文件，确保目标框架是 `net8.0`。

**步骤 2：编写类和方法**
在 `Class1.cs` 文件中编写以下代码：

```csharp
namespace MyNet8Library
{
    public class Calculator
    {
        public int Add(int a, int b)
        {
            return a + b;
        }
    }
}
```

**步骤 3：编译类库**
在项目目录下执行以下命令进行编译：

```sh
dotnet build
```

**步骤 4：在 PowerShell 中调用类库方法**

```powershell
# 加载编译后的 DLL 文件
Add-Type -Path "C:\path\to\MyNet8Library\bin\Debug\net8.0\MyNet8Library.dll"

# 创建类的实例
$calculator = New-Object MyNet8Library.Calculator

# 调用方法
$result = $calculator.Add(3, 5)
Write-Output "3 + 5 的结果是: $result"
```

在这个示例中：

- `Add-Type -Path` 用于加载自定义类库的 DLL 文件。
- `New-Object` 用于创建类的实例对象。
- `$calculator.Add(3, 5)` 调用了 `Calculator` 类的 `Add` 方法。

通过以上步骤，你就可以在 PowerShell 7.5.0 中调用 .NET 8 的类方法了。