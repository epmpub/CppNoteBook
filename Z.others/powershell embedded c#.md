# powershell embedded c#

在 PowerShell 中嵌入 C# 代码是一种强大的方式，可以利用 C# 的功能来扩展 PowerShell 脚本的能力。以下是实现这一目标的详细步骤和示例，涵盖了常见的使用场景。

方法：使用 Add-Type 嵌入 C# 代码

PowerShell 提供了 Add-Type cmdlet，允许你直接在脚本中定义和编译 C# 代码，并调用其功能。以下是具体步骤：

1. 基本嵌入 C# 代码的示例

以下是一个简单的示例，展示如何在 PowerShell 中嵌入 C# 代码并调用它：

powershell

```powershell
# 定义 C# 代码
$csharpCode = @"
using System;

public class Calculator
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}
"@

# 使用 Add-Type 编译 C# 代码
Add-Type -TypeDefinition $csharpCode

# 调用 C# 方法
$result = [Calculator]::Add(5, 3)
Write-Output "5 + 3 = $result"
```

**输出**：

```text
5 + 3 = 8
```

说明：

- **C# 代码定义**：将 C# 代码存储在一个多行字符串（here-string）中。
- **Add-Type**：将 C# 代码编译为 .NET 程序集，并加载到当前 PowerShell 会话中。
- **调用**：使用 [ClassName]::MethodName 的语法调用静态方法。
- 更复杂的 C# 代码

如果需要更复杂的逻辑，比如实例化对象或处理复杂数据，可以扩展 C# 代码。例如：

powershell

```powershell
write-host "Hello World"

$csharpCode = @"
using System;
using System.Collections.Generic;

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }

    public string GetGreeting()
    {
        return string.Format("Hello, my name is {0} and I am {1} years old.", Name, Age);
        // 注意这里 , .net framework version ,c# 6 or 4.5 syntex 
        // string.Format is compatible with all C# versions and achieves the same result by using placeholders ({0}, {1}) for 	         // Name and Age.


    }

    public static List<Person> GetSamplePeople()
    {
        return new List<Person>
        {
            new Person("Alice", 25),
            new Person("Bob", 30)
        };
    }
}
"@

try {
    # Attempt to compile the C# code
    Add-Type -TypeDefinition $csharpCode -Language CSharp
    Write-Host "C# code compiled successfully."

    # Example usage of the compiled code
    $people = [Person]::GetSamplePeople()
    foreach ($person in $people) {
        Write-Host $person.GetGreeting()
    }
}
catch {
    Write-Host "Error compiling C# code: $_"
}

# 编译 C# 代码
Add-Type -TypeDefinition $csharpCode

# 创建实例并调用方法
$person = New-Object Person("Charlie", 28)
$greeting = $person.GetGreeting()
Write-Output $greeting

# 调用静态方法
$people = [Person]::GetSamplePeople()
foreach ($p in $people) {
    Write-Output "$($p.Name) is $($p.Age) years old."
}
```

**输出**：

```text
Hello, my name is Charlie and I am 28 years old.
Alice is 25 years old.
Bob is 30 years old.
```

3. 引用外部程序集

如果 C# 代码需要引用额外的 .NET 程序集（比如 System.Windows.Forms），可以使用 Add-Type 的 -ReferencedAssemblies 参数：

powershell

```powershell
$csharpCode = @"
using System;
using System.Windows.Forms;

public class MessageBoxHelper
{
    public static void ShowMessage(string message)
    {
        MessageBox.Show(message, "PowerShell C# Demo", MessageBoxButtons.OK, MessageBoxIcon.Information);
    }
}
"@

# 编译并引用 System.Windows.Forms
Add-Type -TypeDefinition $csharpCode -ReferencedAssemblies "System.Windows.Forms"

# 调用 C# 方法显示消息框
[MessageBoxHelper]::ShowMessage("Hello from PowerShell and C#!")
```

**效果**：弹出一个 Windows 消息框，显示指定的消息。

4. 错误处理和调试

- **编译错误**：如果 C# 代码有语法错误，Add-Type 会抛出异常，显示详细的编译错误信息。确保检查代码的语法。
- **调试**：可以在 C# 代码中加入 Console.WriteLine 或使用 PowerShell 的 Write-Output 来调试输出。
- **命名冲突**：如果多次运行 Add-Type 定义相同的类，PowerShell 会报错。可以检查类型是否已加载：

powershell

```powershell
if (-not ([System.Management.Automation.PSTypeName]'Calculator').Type) {
    Add-Type -TypeDefinition $csharpCode
}
```

5. 使用 C# 动态编译大型项目

如果 C# 代码较复杂或需要多个文件，可以将代码保存在 .cs 文件中，然后使用 Add-Type 加载：

powershell

```powershell
Add-Type -Path "C:\Path\To\YourCode.cs"
```

6. 性能注意事项

- **编译开销**：Add-Type 会在每次会话中编译代码。如果脚本需要频繁运行，考虑将编译后的程序集保存为 DLL 文件并加载：

powershell

```powershell
Add-Type -TypeDefinition $csharpCode -OutputAssembly "MyAssembly.dll"
# 后续加载 DLL
Add-Type -Path "MyAssembly.dll"
```

- **命名空间**：为避免冲突，建议在 C# 代码中使用命名空间：

csharp

```csharp
namespace MyNamespace {
    public class MyClass {
        // ...
    }
}
```

7. 实际应用场景

- **文件操作**：使用 C# 的 System.IO 进行高效的文件读写。
- **网络请求**：通过 System.Net.Http 实现复杂的 HTTP 请求。
- **GUI 界面**：结合 System.Windows.Forms 或 System.Windows.Controls 创建图形界面。
- **性能优化**：将计算密集型任务交给 C# 处理，提升性能。

**限制和注意事项**

- **PowerShell 版本**：某些功能可能依赖于 PowerShell 版本（如 PowerShell 5.1 或 PowerShell Core 7+）。
- **.NET 框架**：PowerShell 5.1 默认使用 .NET Framework，而 PowerShell Core 使用 .NET Core。确保 C# 代码与目标 .NET 运行时兼容。
- **权限**：某些操作（如访问系统资源）可能需要提升的权限。
- 替代方法

如果需要更复杂的 C# 项目，考虑以下替代方案：

- **创建 DLL**：在 Visual Studio 中创建 C# 类库，编译为 DLL，然后在 PowerShell 中通过 Add-Type -Path 加载。
- **PowerShell 模块**：将 C# 代码封装为 PowerShell 模块，方便复用。

总结

通过 Add-Type，你可以在 PowerShell 中嵌入和执行 C# 代码，结合两者的优势来实现复杂功能。建议从简单示例开始，逐步扩展到更复杂的场景，同时注意错误处理和性能优化。如果需要更具体的代码示例或针对特定场景的帮助，请提供更多细节，我可以进一步定制解决方案！