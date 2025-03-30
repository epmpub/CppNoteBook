# PowerShell SendKey

```powershell
Add-Type -AssemblyName System.Windows.Forms
$vpnclipath = "C:\Program Files (x86)\Cisco\Cisco AnyConnect Secure Mobility 
Client\vpncli.exe"
$VPN = "my.vpn.com"
$group = "1"
$user = "DOMAIN\$env:username"
$securepass = Import-Clixml -Path 'C:\Temp\VPN.xml'
$password = [Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($securepass))
$accept? = "y"



###Close all Cisco Apps###
Stop-Process -Name vpnagent.exe -Force
Stop-Process -Name vpnui.exe -Force
Start-Sleep -m 2000



###Disconnect VPN###
Function VPNDisconnect()
{
Start-Process -FilePath $vpnclipath -ArgumentList "disconnect"
}
VPNDisconnect
Start-Sleep -m 2000

###Close all Cisco Apps###
Stop-Process -Name vpnagent.exe -Force
Stop-Process -Name vpnui.exe -Force
Start-Sleep -m 2000

###Connect to VPN###
Function VPNConnect()
{
Start-Process -FilePath $vpnclipath -ArgumentList "connect $VPN"
}

VPNConnect

Start-Sleep -m 2000
[System.Windows.Forms.SendKeys]::SendWait("$group{Enter}")
Start-Sleep -m 2000
[System.Windows.Forms.SendKeys]::SendWait("$user{Enter}")
Start-Sleep -m 2000
[System.Windows.Forms.SendKeys]::SendWait("$password{Enter}")
Start-Sleep -m 2000
[System.Windows.Forms.SendKeys]::SendWait("$accept?{Enter}")
Start-Sleep -m 2000
```

