下面给你 **Windows 11 用 PowerShell 强制开启 / 关闭 Wi-Fi 随机 MAC 地址** 的**可落地方案**。我按 **生效层级** 从「最可靠」到「尽量而为」给你。

> 适用于 **PowerShell 5.1 / 7**
> ⚠ 需要 **管理员权限**

------

# ✅ 方案一（最可靠）：按 Wi-Fi SSID 强制开启 / 关闭（注册表）

> **Windows 11 实际是“按 Wi-Fi 配置文件”控制随机 MAC**

## 1️⃣ 查看所有 Wi-Fi 配置文件

```powershell
$base = "HKLM:\SOFTWARE\Microsoft\WlanSvc\Interfaces"

Get-ChildItem $base -Recurse -ErrorAction SilentlyContinue |
ForEach-Object {
    $p = Get-ItemProperty $_.PSPath -ErrorAction SilentlyContinue
    if ($p.PSObject.Properties.Name -contains "ProfileName") {
        [PSCustomObject]@{
            SSID = $p.ProfileName
            RandomMacEnabled = $p.RandomMacEnabled
            Path = $_.PSPath
        }
    }
} | Format-Table -AutoSize
```

------

## 2️⃣ **强制开启随机 MAC**

```powershell
$ssid = "YourWiFiName"

Get-ChildItem "HKLM:\SOFTWARE\Microsoft\WlanSvc\Interfaces" -Recurse |
ForEach-Object {
    $p = Get-ItemProperty $_.PSPath -ErrorAction SilentlyContinue
    if ($p.ProfileName -eq $ssid) {
        Set-ItemProperty $_.PSPath -Name RandomMacEnabled -Value 1 -Type DWord
    }
}
```

------

## 3️⃣ **强制关闭随机 MAC**

```powershell
$ssid = "YourWiFiName"

Get-ChildItem "HKLM:\SOFTWARE\Microsoft\WlanSvc\Interfaces" -Recurse |
ForEach-Object {
    $p = Get-ItemProperty $_.PSPath -ErrorAction SilentlyContinue
    if ($p.ProfileName -eq $ssid) {
        Set-ItemProperty $_.PSPath -Name RandomMacEnabled -Value 0 -Type DWord
    }
}
```

------

## 4️⃣ 立即生效（断开 / 重连）

```powershell
netsh wlan disconnect
Start-Sleep 2
netsh wlan connect name="YourWiFiName"
```

------

# ⚠ 方案二（不稳定）：驱动层（网卡高级属性）

> **是否成功完全取决于网卡驱动**

## 查看支持项

```powershell
Get-NetAdapterAdvancedProperty -Name "Wi-Fi" |
Where-Object DisplayName -Match "Random|MAC"
```

### 尝试开启

```powershell
Set-NetAdapterAdvancedProperty `
  -Name "Wi-Fi" `
  -DisplayName "Random Hardware Addresses" `
  -DisplayValue "Enabled"
```

### 尝试关闭

```powershell
Set-NetAdapterAdvancedProperty `
  -Name "Wi-Fi" `
  -DisplayName "Random Hardware Addresses" `
  -DisplayValue "Disabled"
```

> ❌ 很多 Intel / Realtek 驱动 **根本不支持这一项**

------

# 🧪 一键脚本（自动处理 + 验证）

```powershell
$ssid = "YourWiFiName"

$found = $false
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\WlanSvc\Interfaces" -Recurse |
ForEach-Object {
    $p = Get-ItemProperty $_.PSPath -ErrorAction SilentlyContinue
    if ($p.ProfileName -eq $ssid) {
        Set-ItemProperty $_.PSPath -Name RandomMacEnabled -Value 1 -Type DWord
        $found = $true
        Write-Host "✔ Random MAC enabled for $ssid"
    }
}

if (-not $found) {
    Write-Warning "SSID not found: $ssid"
}
```

------

# 📌 验证是否成功

```powershell
Get-NetAdapter -Name "Wi-Fi" | Select MacAddress
```

或切换到**另一个 Wi-Fi**再切回，看 MAC 是否变化。

------

# 🚨 注意事项（非常重要）

- 🔐 **公司 / 学校 Wi-Fi（802.1X）可能强制关闭**
- 🧩 **组策略可能覆盖注册表**
- 🔄 必须 **断开重连** 才会重新生成 MAC
- 🛑 修改注册表有风险（生产环境请先导出）

------

## 如果你愿意，我可以再帮你：

- ✅ **对所有 Wi-Fi SSID 批量开启**
- 🏢 **检测是否被组策略禁用**
- 📦 **封装成 `.ps1` + 参数（Enable/Disable）**
- 🧪 **输出前后 MAC 对比报告**

告诉我你想要哪一个版本。