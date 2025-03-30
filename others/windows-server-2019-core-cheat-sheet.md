------

# windows-server-2019-core-cheat-sheet

### Table of Contents

- [Windows Server 2019 Core Cheat Sheet](https://wiki.chotaire.net/windows-server-2019-core-cheat-sheet#windows_server_2019_core_cheat_sheet)
  - [Enable Remote Desktop](https://wiki.chotaire.net/windows-server-2019-core-cheat-sheet#enable_remote_desktop)
  - [Disable Password Complexity](https://wiki.chotaire.net/windows-server-2019-core-cheat-sheet#disable_password_complexity)
  - [Disable Password Age Restrictions](https://wiki.chotaire.net/windows-server-2019-core-cheat-sheet#disable_password_age_restrictions)
  - [Add App Compatibility FOD](https://wiki.chotaire.net/windows-server-2019-core-cheat-sheet#add_app_compatibility_fod)
  - [Install .NET Framework on Windows Server Core](https://wiki.chotaire.net/windows-server-2019-core-cheat-sheet#install_net_framework_on_windows_server_core)
  - [Change display language](https://wiki.chotaire.net/windows-server-2019-core-cheat-sheet#change_display_language)

![img](https://wiki.chotaire.net/_media/public:windows_server_2019_logo.svg.png?w=200&tok=f3db10)

# Windows Server 2019 Core Cheat Sheet

## Enable Remote Desktop

In ***cmd.exe***, enter the following command to **enable** Remote Desktop connections for this host:

```
cscript %windir%\system32\scregedit.wsf /ar 0
```

**Verify** the setting with:

```
cscript %windir%\system32\scregedit.wsf /ar /v
```

Later, if you would like to **disable** Remote Desktop connections:

```
cscript %windir%\system32\scregedit.wsf /ar 1
```

Alternatively, Remote Desktop can also be switched on and off using **sconfig**.

If you install the **[App Compatibility FOD](https://wiki.chotaire.net/windows-server-2019-core-cheat-sheet#add_app_compatibility_fod)** as well, you can easily transfer files to your Server Core instance by establishing a Remote Desktop connection with one of your local drives shared via RDP. Open ***explorer.exe*** on your Windows Server Core 2019 to see your PC's drive.

## Disable Password Complexity

In a ***powershell***, use the following commands:

```
secedit /export /cfg c:\secpol.cfg
(gc C:\secpol.cfg).replace("PasswordComplexity = 1", "PasswordComplexity = 0") | Out-File C:\secpol.cfg
secedit /configure /db c:\windows\security\local.sdb /cfg c:\secpol.cfg /areas SECURITYPOLICY
rm -force c:\secpol.cfg -confirm:$false
```

## Disable Password Age Restrictions

In a ***powershell***, use the following commands.

To disable Password Age restrictions for a **single user**:

```
Set-LocalUser -Name "Administrator" -PasswordNeverExpires 1
```

To disable Password Age restrictions for **all users**:

```
net accounts /maxpwage:unlimited
```

## Add App Compatibility FOD

The App Compatibility FOD allows you to run a couple more GUI applications that would otherwise refuse to run. Also it will install a few tools you know from Desktop Experience. Download the App Compatibility FOD ISO from here: **[Windows Server 2019 Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019)**.

You might need this. This will install:

- Microsoft Management Console (mmc.exe)
- Event Viewer (Eventvwr.msc)
- Performance Monitor (PerfMon.exe)
- Resource Monitor (Resmon.exe)
- Device Manager (Devmgmt.msc)
- File Explorer (Explorer.exe)
- Windows PowerShell (Powershell_ISE.exe)
- Disk Management (Diskmgmt.msc)

Optional: If you need to **mount** it within Windows Server 2019, type this in a ***powershell\***:

```
Mount-DiskImage -ImagePath drive_letter:\folder_where_ISO_is_saved\ISO_filename.iso
```

To install App Compatibility FOD type this:

```
DISM /Online /Add-Capability /CapabilityName:"ServerCore.AppCompatibility~~~~0.0.1.0" /Source:drive_letter_of_mounted_ISO: /LimitAccess
```

If really necessary, you can also install **Internet Explorer 11** (has a few limitations compared to Desktop Experience):

```
Dism /online /add-package:drive_letter_of_mounted_iso:"Microsoft-Windows-InternetExplorer-Optional-Package~31bf3856ad364e35~amd64~~.cab"
```

## Install .NET Framework on Windows Server Core

Note: Surprise, this is already included on Hyper-V Server 2019.

- Download from here: **[.NET Framework 4.7.2 Offline Installer](https://support.microsoft.com/de-de/help/4054530/microsoft-net-framework-4-7-2-offline-installer-for-windows)**
- Copy to your Windows Server Core instance (e.g. using Remote Desktop via a shared drive)
- Run the installer using the **/q** flag. It is required because we do not have a full GUI.

```
NDP472-KB4054530-x86-x64-AllOS-ENU.exe /q /norestart
```

You will see absolutely no progress, so give it some time to install before you reboot or shutdown. You can get an idea by watching **Task Manager** (CTRL + ALT + END, then select Task Manager in a RDP session). Remember to install **Windows Updates** via **sconfig** afterwards.

## Change display language

First, get the language pack DVD that is compatible with your version of Windows Server 2019. At the time of this writing, the language pack for the current release of Windows Server 2019 is available **[right here](https://software-download.microsoft.com/download/pr/17763.1.180914-1434.rs5_release_SERVERLANGPACKDVD_OEM_MULTI.iso)**. Should it get outdated, hopefully Microsoft will update [this](https://software-download.microsoft.com/download/pr/17763.1.180914-1434.rs5_release_SERVERLANGPACKDVD_OEM_MULTI.iso) document with the updated link.

In a ***cmd.exe\*** as an Administrator, run the following command, then navigate to the mounted DVD and install the languages of choice:

```
lpksetup
```

Now we need to head over to Microsoft to see the **[table of geographical locations](https://docs.microsoft.com/en-us/windows/desktop/intl/table-of-geographical-locations)**. Take note of the decimal value for your desired location.

In a ***powershell*** (**run as Administrator**), type the following command to change language and regional settings for the current user (in this example to German).

```
Set-Culture de-DE
Set-WinSystemLocale de-DE
Set-WinHomeLocation -GeoId 94
Set-WinUserLanguageList de-DE -Force
Set-WinUILanguageOverride de-DE
```

…followed by a reboot. Don't ask me why, sometimes you have to do this twice (and reboot twice) before you see actual changes. Up next, we are going to open the **Control panel** (requires App Compatibility FOD to be installed):

```
control.exe
```

Navigate to **Clock and Region → Region** and open it. Then go to **Administrative → Language for non-unicode programs** and click on **“Change system locale…“** if that one is still not set to your language of choice.

Click on “**copy settings**” to see a tidy list and verify everything is set correctly. If you would like to switch this language to **Welcome Screen, system accounts and new user accounts**, [x] them all and click on Ok, then reboot once more.



------



```powershell
#Requires -Version 5.1

<#
.SYNOPSIS
    Set the minium password length or age for local accounts.
.DESCRIPTION
    Set the minium password length or age for local accounts.
.EXAMPLE
     -Length 14
    Set the minium password length for local accounts.
.EXAMPLE
     -Length 14 -MinAge 30 -MaxAge 42
    Set the minium password length, minium age, maximum age for local accounts.
.EXAMPLE
    PS C:\> Set-MiniumPasswordRequirements.ps1 -Length 14 -Age 42
    Set the minium password length and age for local accounts.
.OUTPUTS
    None
.NOTES
    Minimum OS Architecture Supported: Windows 10, Windows Server 2016
    Release Notes:
    Initial Release
By using this script, you indicate your acceptance of the following legal terms as well as our Terms of Use at https://www.ninjaone.com/terms-of-use.
    Ownership Rights: NinjaOne owns and will continue to own all right, title, and interest in and to the script (including the copyright). NinjaOne is giving you a limited license to use the script in accordance with these legal terms. 
    Use Limitation: You may only use the script for your legitimate personal or internal business purposes, and you may not share the script with another party. 
    Republication Prohibition: Under no circumstances are you permitted to re-publish the script in any script library or website belonging to or under the control of any other software provider. 
    Warranty Disclaimer: The script is provided “as is” and “as available”, without warranty of any kind. NinjaOne makes no promise or guarantee that the script will be free from defects or that it will meet your specific needs or expectations. 
    Assumption of Risk: Your use of the script is at your own risk. You acknowledge that there are certain inherent risks in using the script, and you understand and assume each of those risks. 
    Waiver and Release: You will not hold NinjaOne responsible for any adverse or unintended consequences resulting from your use of the script, and you waive any legal or equitable rights or remedies you may have against NinjaOne relating to your use of the script. 
    EULA: If you are a NinjaOne customer, your use of the script is subject to the End User License Agreement applicable to you (EULA).
.COMPONENT
    LocalUserAccountManagement
#>

[CmdletBinding()]
param (
    [Parameter()]
    [ValidateRange(0, 14)]
    [int]
    $Length,
    [Parameter()]
    [ValidateRange(0, 998)]
    [int]
    $MinAge,
    [Parameter()]
    [ValidateRange(0, 999)]
    [int]
    $MaxAge
)

begin {
    function Test-IsElevated {
        $id = [System.Security.Principal.WindowsIdentity]::GetCurrent()
        $p = New-Object System.Security.Principal.WindowsPrincipal($id)
        if ($p.IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator))
        { Write-Output $true }
        else
        { Write-Output $false }
    }
    function Get-LocalPasswordPolicy {
        param ()
        $Result = [PSCustomObject]@{
            MinimumLength = 0
            MaximumAge    = 0
            MinimumAge    = 0
        }
        $(net.exe accounts) -split "\n" | ForEach-Object {
            $Line = $_ -split ":"
            if ($_ -like "Minimum password length*") {
                $Result.MinimumLength = "$($Line[1])".Trim(' ')
            }
            if ($_ -like "Maximum password age (days)*") {
                $Result.MaximumAge = "$($Line[1])".Trim(' ')
            }
            if ($_ -like "Minimum password age (days)*") {
                $Result.MinimumAge = "$($Line[1])".Trim(' ')
            }
        }
        $Result
    }

    $NetExeError = $false
}
process {
    if (-not (Test-IsElevated)) {
        Write-Error -Message "Access Denied. Please run with Administrator privileges."
        exit 1
    }
    # Get Current localhost password policy settings
    $CurrentSettings = Get-LocalPasswordPolicy
    
    if ($PSBoundParameters.ContainsKey("Length") -and -not ($CurrentSettings.MinimumLength -like $NewSettings.MinimumLength -or $CurrentSettings.MinimumLength -eq $NewSettings.MinimumLength)) {
        Write-Host "Changing Minimum Password Length from $($CurrentSettings.MinimumLength) to $Length"
        net.exe accounts /minpwlen:$Length
    }
    if ($PSBoundParameters.ContainsKey("MaxAge") -and
        -not (
            $(if ($CurrentSettings.MaximumAge -like "unlimited") { 0 }else { $CurrentSettings.MaximumAge }) -like
            $(if ($NewSettings.MaximumAge -like "unlimited") { 0 }else { $NewSettings.MaximumAge })
        )) {
        Write-Host "Changing Maximum Password Age from $($CurrentSettings.MaximumAge) to $MaxAge"
        if ($MaxAge -gt 0) {
            net.exe accounts /maxpwage:$MaxAge
        }
        else {
            net.exe accounts /maxpwage:unlimited
        }
    }
    if ($PSBoundParameters.ContainsKey("MinAge") -and -not ($CurrentSettings.MinimumAge -like $NewSettings.MinimumAge -or $CurrentSettings.MinimumAge -eq $NewSettings.MinimumAge)) {
        Write-Host "Changing Minimum Password Age from $($CurrentSettings.MinimumAge) to $MinAge"
        net.exe accounts /minpwage:$MinAge
    }

    # Get New localhost password policy settings and check if anything changed
    $NewSettings = Get-LocalPasswordPolicy
    
    if ($PSBoundParameters.ContainsKey("Length") -and ($PSBoundParameters['Length'] -notlike $NewSettings.MinimumLength)) {
        $NetExeError = $true
        Write-Host "Minimum Length was not set correctly."
    }
    if ($PSBoundParameters.ContainsKey("MaxAge") -and ($PSBoundParameters['MaxAge'] -notlike $(if ($NewSettings.MaximumAge -like "unlimited") { 0 }else { $NewSettings.MaximumAge }))) {
        $NetExeError = $true
        Write-Host "Maximum Age was not set correctly."
    }
    if ($PSBoundParameters.ContainsKey("MinAge") -and ($PSBoundParameters['MinAge'] -notlike $NewSettings.MinimumAge)) {
        $NetExeError = $true
        Write-Host "Minimum Age was not set correctly."
    }
    if ($NetExeError) {
        exit 1
    }
}
end {}
```

