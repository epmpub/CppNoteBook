# Windows Server 2019 security policy 设置



# #1 set password policy

```bat
@echo off
echo [version]>gp.inf
echo signature="$CHICAGO$">>gp.inf
echo [System Access]>>gp.inf
echo MinimumPasswordAge = 0 >>gp.inf
echo MaximumPasswordAge = 90 >>gp.inf
echo MinimumPasswordLength = 12 >>gp.inf
echo PasswordComplexity = 1 >>gp.inf
echo LockoutBadCount = 5 >>gp.inf
echo LockoutDuration = 30 >>gp.inf
echo PasswordHistorySize = 5  >>gp.inf
secedit /configure /db gp.sdb /cfg gp.inf
del /f /q gp.inf gp.sdb gp.jfm




```

```powershell
gpedit.msc
```



![image-20231207155321705](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231207155321705.png)

## #2 rename Administrator

![image-20231207160050827](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231207160050827.png)

```powershell
Rename-LocalUser -Name "Administrator" -NewName "clouduser"
```



## #3 disable guest account

```powershell
net user /guest /active:no
Get-LocalUser Guest | Disable-LocalUser

```

## #4 log setting

```powershell
Limit-EventLog -LogName Application -MaximumSize 50Mb -OverflowAction OverwriteAsNeeded
Limit-EventLog -LogName Security -MaximumSize 50Mb -OverflowAction OverwriteAsNeeded
Limit-EventLog -LogName System -MaximumSize 50Mb -OverflowAction OverwriteAsNeeded
```

## #5 set interact logon setting

```
系统在每次登陆的时候会显示上一次登陆的用户的用户名，入侵者很可能会根据被显示出来的用户名获取登陆密码并且非法进入系统。此项的手工修复方法如下：
运行注册表编辑器regedit.exe，修改键 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\system 中的值 dontdisplaylastusername， dontdisplaylastusername＝0 为显示显示上一次登陆的用户的用户名， dontdisplaylastusername＝1 为不显示显示上一次登陆的用户的用户名
```

![image-20231207162718875](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231207162718875.png)

```powershell
if((Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\).dontdisplaylastusername -eq 0) {Set-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\ -Name dontdisplaylastusername -Value 1}
```



## #6 WSUS Server

```
$Tgg = "YourTargetGroup"
$Wup = "http://yourprimarywsus.contoso.local:8530"
$Wur = "http://yourprimaryorsecudairywsus.contoso.local:8530"

#Stop all before the magic can happen
stop-service wuauserv -Force
stop-service bits -Force
stop-service usosvc -Force
stop-service cryptsvc -Force

#Removing all old 
Remove-item -Path 'C:\windows\SoftwareDistribution' -Recurse -Force -Confirm:$false -ErrorAction SilentlyContinue
Remove-item -Path 'C:\windows\SoftwareDistribution\Datastore' -Recurse -Force -Confirm:$false -ErrorAction SilentlyContinue
Remove-item -Path 'C:\windows\SoftwareDistribution\Download' -Recurse -Force -Confirm:$false -ErrorAction SilentlyContinue

#Force set WU client settings
New-Item –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" -ErrorAction SilentlyContinue
New-ItemProperty –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" –Name TargetGroup -Value $Tgg -PropertyType "String" -Force -ErrorAction SilentlyContinue
New-ItemProperty –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" –Name WUServer -Value $Wup -PropertyType "String" -Force -ErrorAction SilentlyContinue
New-ItemProperty –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" –Name WUStatusServer -Value $Wup -PropertyType "String" -Force -ErrorAction SilentlyContinue
New-ItemProperty –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" –Name UpdateServiceUrlAlternate -Value $Wur -PropertyType "String" -Force -ErrorAction SilentlyContinue

New-ItemProperty –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" –Name TargetGroupEnabled -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue
New-ItemProperty –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" –Name DoNotConnectToWindowsUpdateInternetLocations -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue

New-Item –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -ErrorAction SilentlyContinue
New-ItemProperty –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" –Name UseWUServer -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue
New-ItemProperty –Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" –Name NoAutoUpdate -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue


#Start all necessary services
start-service wuauserv 
start-service bits
start-service usosvc
start-service cryptsvc 

#Detect and reset auth
start-sleep -s 60
cmd /c "wuauclt /resetauthorization /detectnow"

$updatesession =  [activator]::CreateInstance([type]::GetTypeFromProgID("Microsoft.Update.Session",$env:COMPUTERNAME)) 
$updatesearcher = $updatesession.CreateUpdateSearcher() 
try{
$searchresult = $updatesearcher.Search("IsInstalled=1")
} Catch {}
if(!$searchresult){
    stop-service wuauserv -force
    Start-Service wuauserv
    start-sleep -s 60 

    $updatesession =  [activator]::CreateInstance([type]::GetTypeFromProgID("Microsoft.Update.Session",$env:COMPUTERNAME)) 
    $updatesearcher = $updatesession.CreateUpdateSearcher() 
    $searchresult = $updatesearcher.Search("IsInstalled=1")

}

start-sleep -s 60
cmd /c "wuauclt /reportnow"
```

## #7 Screensave setting?

​	

```powershell
New-ItemProperty -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\ -Name InactivityTimeoutSecs -Type DWORD -Value 600
```

## #8 关闭防火墙

```
netsh advfirewall set all state off
```

## #9 禁用防火墙Disable Firewall

```powershell
Set-NetFirewallProfile -Enabled False
```

## #10 关闭Windows Defender 防病毒

```
Set-MpPreference -DisableRealtimeMonitoring $true
```

## #11 电源选项

​	

```
powercfg /SETACTIVE 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c
```

## #12 安装open SSH 用windows server 自带的，还是用第3方？

​	

```
Get-WindowsCapability -Online | ? Name -like ‘OpenSSH*’
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service -Name sshd
```

#13 Close telemetry 

```
Get-ScheduledTask  -TaskPath "\Microsoft\Windows\Application Experience\" | Disable-ScheduledTask
Get-ScheduledTask -TaskName 'StartupAppTask' |Enable-ScheduledTask

 Get-ScheduledTask -TaskName 'Consolidator' |Disable-ScheduledTask
 
 takeown /f C:\Windows\System32\CompatTelRunner.exe
 icacls C:\Windows\System32\CompatTelRunner.exe /deny system;F
 icacls C:\Windows\System32\CompatTelRunner.exe /deny users:F
 
 
 
```

#14 修改TCP参数

```powershell
New-ItemProperty  HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\ -Name MaxUserPcrt -Type DWORD -Value 65534
```



#15 system uptime 获取开机时间#

```powershell
 if (((Get-Date)-(gcim Win32_OperatingSystem).LastBootUpTime).Days > 30)
 {"Greate Than 30 Days"}
 else
 {"update time is OK"}
```

#16 安装pstools ，到目录 systools

```
$ProgressPreference = 'SilentlyContinue'
$targetDirectory = 'c:\systools'
if (-not(Test-Path $targetDirectory))
{
  New-Item -Path $targetDirectory -ItemType Directory | Out-Null
}
curl -Uri https://download.sysinternals.com/files/SysinternalsSuite.zip -OutFile $targetDirectory\1.zip
Expand-Archive -Path $targetDirectory\1.zip -DestinationPath $targetDirectory -Force
Remove-Item -Path $targetDirectory\1.zip -Force
```

