## Windows配置为 NTP 时间服务器

##### 配置步骤

### 第一步：配置服务器同步外部 NTP 源

以管理员身份打开命令提示符，执行：

```cmd
w32tm /config /manualpeerlist:"time.windows.com,0x8 time.nist.gov,0x8" /syncfromflags:manual /reliable:YES /update
```

**参数说明：**

- `manualpeerlist` - 指定上游 NTP 服务器
- `0x8` - 表示使用客户端模式
- `reliable:YES` - 设置为可靠的时间源

### 第二步：配置为 NTP 服务器

```cmd
w32tm /config /update
```

### 第三步：启用 NTP 服务器功能

使用注册表编辑器配置：

1. 按 `Win + R`，输入 `regedit`
2. 导航到：`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpServer`
   - 将 **Enabled** 设置为 `1`
3. 导航到：`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\Config`
   - 将 **AnnounceFlags** 设置为 `5`

### 第四步：配置防火墙

允许 NTP 流量（UDP 123 端口）：

```cmd
netsh advfirewall firewall add rule name="NTP Server" dir=in action=allow protocol=UDP localport=123
```

或通过图形界面：

- 打开 **Windows 防火墙高级设置**
- 新建入站规则
- 协议类型：UDP
- 本地端口：123

### 第五步：重启 Windows Time 服务

```cmd
net stop w32time
net start w32time
```

### 第六步：强制同步并验证

```cmd
w32tm /resync /rediscover
w32tm /query /status
w32tm /query /configuration
```

## 完整的 PowerShell 脚本

您也可以使用这个脚本自动配置：

```powershell
# 配置 NTP 服务器
w32tm /config /manualpeerlist:"time.windows.com,0x8 time.nist.gov,0x8" /syncfromflags:manual /reliable:YES /update

# 启用 NTP 服务器
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpServer" -Name "Enabled" -Value 1

# 设置为可靠时间源
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\W32Time\Config" -Name "AnnounceFlags" -Value 5

# 配置防火墙规则
New-NetFirewallRule -DisplayName "NTP Server" -Direction Inbound -Protocol UDP -LocalPort 123 -Action Allow

# 重启服务
Restart-Service w32time

# 强制同步
w32tm /resync /rediscover

# 显示状态
w32tm /query /status
```

## 客户端配置示例

### Windows 客户端

```cmd
w32tm /config /manualpeerlist:"你的服务器IP" /syncfromflags:manual /update
net stop w32time && net start w32time
w32tm /resync
```

### Linux 客户端

编辑 `/etc/ntp.conf` 或 `/etc/chrony/chrony.conf`：

```
server 你的服务器IP iburst
```

### 网络设备

在设备配置中添加 NTP 服务器地址即可。

## 验证和监控

**检查服务器状态：**

```cmd
w32tm /query /status
w32tm /query /peers
```

**查看日志：**

- 事件查看器 → Windows 日志 → 系统
- 查找来源为 "Time-Service" 的事件

**测试客户端连接：**

```cmd
w32tm /stripchart /computer:你的服务器IP /samples:5
```

## 常见问题

**如果时间不同步：**

```cmd
w32tm /resync /force
```

**如果服务无法启动：**

```cmd
w32tm /unregister
w32tm /register
net start w32time
```

需要其他帮助吗？比如域环境下的配置或故障排查？