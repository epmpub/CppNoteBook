## 文件系统监控工具

​     今天给大家介绍一个简单的文件系统监控管理命令。windows系统不像Linux系统，‘Linux下一切皆文件’。

但windows下对文件的操作，很减少很多重复的操作。

![img](https://pica.zhimg.com/80/v2-88df96a0822eef2b843b3086ac83858f_1440w.png?source=d16d100b)

​													（软件架构）

## **0.使用场景：**

- 软件工程中，构建系统之前，会做系统分析和架构设计，是软件工程最重要的部分。在构建完后，进入运维阶段，这个阶段是重复工作，但是非常花费时间，需要一套流程自动化的系统，来完成部署、测试。
- 设计一套运行的流程，如数据的处理流程，例如，可以在客户端将文件上传到服务器，在服务器上对单个文件进行处理，比如抽取、合并。

## **1. 工具介绍：**

File System Watcher ，别名fswt是一个简单的命令行工具，用来实时查看当前目录的exe文件变化情况，便且执行指定的命令。

## **2. 安装方式：**

在命令行下面输入安装fswt

```
 winget install fswt
```

![img](https://picx.zhimg.com/80/v2-5bf5fa14c95898095cdc15339c0f0eb2_1440w.jpg?source=d16d100b)												winget安装

## **3.命令行使用:**

在rust的生成目录下\target\debug 运行如下命令：

```
PS C:\Users\HelloWin\rust_project\win_rust_1\target\debug> fswt "xcopy *.exe \\192.168.200.4\Users\HelloWin\Desktop\dev /Y"
```

以上命令行意思为： 只要exe文件发生变化，就将当前目录下的exe文件copy到远程的共享目录中

这个远程目录为另外一台VM，专门用来测试exe文件。

![img](https://picx.zhimg.com/80/v2-b9f273a340177d6c27d168830da08769_1440w.jpg?source=d16d100b)													VM测试



远程测试VM

这个只是最简单的应用，可以用scp/ssh 用来做Linux主机的远程测试

以下是软件的极简使用说明。

有问题欢迎评论区留言，一起交流。

源码我放在github，目前功能是最简化的，可以PR或者提交Issues,我会回复。

![img](https://pica.zhimg.com/80/v2-dfdbceeb7c2abde8c335846782a4b74d_1440w.jpg?source=d16d100b)													命令行帮助



Filesystem Watcher Sample代码：

```
using System;
using System.IO;

namespace MyNamespace
{
    class MyClassCS
    {
        static void Main()
        {
            using var watcher = new FileSystemWatcher(@"C:\path\to\folder");

            watcher.NotifyFilter = NotifyFilters.Attributes
                                 | NotifyFilters.CreationTime
                                 | NotifyFilters.DirectoryName
                                 | NotifyFilters.FileName
                                 | NotifyFilters.LastAccess
                                 | NotifyFilters.LastWrite
                                 | NotifyFilters.Security
                                 | NotifyFilters.Size;

            watcher.Changed += OnChanged;
            watcher.Created += OnCreated;
            watcher.Deleted += OnDeleted;
            watcher.Renamed += OnRenamed;
            watcher.Error += OnError;

            watcher.Filter = "*.txt";
            watcher.IncludeSubdirectories = true;
            watcher.EnableRaisingEvents = true;

            Console.WriteLine("Press enter to exit.");
            Console.ReadLine();
        }

        private static void OnChanged(object sender, FileSystemEventArgs e)
        {
            if (e.ChangeType != WatcherChangeTypes.Changed)
            {
                return;
            }
            Console.WriteLine($"Changed: {e.FullPath}");
        }

        private static void OnCreated(object sender, FileSystemEventArgs e)
        {
            string value = $"Created: {e.FullPath}";
            Console.WriteLine(value);
        }

        private static void OnDeleted(object sender, FileSystemEventArgs e) =>
            Console.WriteLine($"Deleted: {e.FullPath}");

        private static void OnRenamed(object sender, RenamedEventArgs e)
        {
            Console.WriteLine($"Renamed:");
            Console.WriteLine($"    Old: {e.OldFullPath}");
            Console.WriteLine($"    New: {e.FullPath}");
        }

        private static void OnError(object sender, ErrorEventArgs e) =>
            PrintException(e.GetException());

        private static void PrintException(Exception? ex)
        {
            if (ex != null)
            {
                Console.WriteLine($"Message: {ex.Message}");
                Console.WriteLine("Stacktrace:");
                Console.WriteLine(ex.StackTrace);
                Console.WriteLine();
                PrintException(ex.InnerException);
            }
        }
    }
}
```

参考：

1. 项目地址：[epmpub/fswt: FileSystem Watcher For Windows 10 (github.com)](https://link.zhihu.com/?target=https%3A//github.com/epmpub/fswt)
2. FileSystemWatcher API参考 [https://learn.microsoft.com/zh-cn/dotnet/api/system.io.filesystemwatcher?view=netframework-4.8.1]