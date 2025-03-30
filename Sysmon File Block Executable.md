# Sysmon File Block Executable



案例：

![image-20231218123358583](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231218123358583.png)

![image-20231218123448932](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231218123448932.png)

![image-20231218144453945](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231218144453945.png)

```xml
<Sysmon schemaversion="4.90">
  <!-- Capture all hashes -->
  <HashAlgorithms>*</HashAlgorithms>
  <EventFiltering>
    <!-- Do not log process termination -->
    <ProcessTerminate onmatch="include" />

    <NetworkConnect onmatch="exclude">
      <Image condition="end with">iexplore.exe</Image>
      <Image condition="end with">svchost.exe</Image>
    </NetworkConnect>

    <DnsQuery onmatch="exclude">
      <Image condition="end with">Sysmon64.exe</Image>
      <Image condition="end with">svchost.exe</Image>
    </DnsQuery>
	
	<FileBlockExecutable onmatch="include">
      <TargetFilename condition="contains all">C:\Users;Downloads</TargetFilename>
    </FileBlockExecutable>
	
	<FileExecutableDetected onmatch="include">
    <TargetFilename condition="begin with">C:\ProgramData\</TargetFilename>
        <TargetFilename condition="begin with">C:\Users\</TargetFilename>
    </FileExecutableDetected>
	

  </EventFiltering>
</Sysmon>
```

# Sysmon

Windows系统健康管理系统

想要知道所有PC,有没有异常情况吗?为什么不做一个检查呢?这样,我们可以知道,是否有一些"弱鸡"在我们的网络里面?    
这个实用程序对任何启动监视器的自动启动位置都有最全面的了解，它向您展示了在系统启动或登录期间配置要运行的程序，以及当您启动各种内置的Windows应用程序(如Internet Explorer、Explorer和媒体播放器)时。这些程序和驱动程序包括你的启动文件夹、Run、RunOnce和其他注册表项中的程序和驱动程序。Autoruns报告Explorer外壳扩展、工具栏、浏览器助手对象、Winlogon通知、自动启动服务等等。Autoruns远远超出了其他自动启动工具。Autoruns的Hide Signed Microsoft Entries选项可以帮助你放大已经添加到你的系统的第三方自动启动image，它支持查看为系统上配置的其他帐户配置的自动启动image.你可能会惊讶于有多少可执行文件会被自动启动!现在你所掌控的不是一台电脑,而是整个网络!
主要功能

- 启动时执行的exe文件
- 服务
- 计划任务
- 驱动程序
- Logon启动项目
- Explorer 插件
- 进程启动、停止
- DNS解析记录

可以将记录发送到es或者阿里sls进行存储分析和展示。

