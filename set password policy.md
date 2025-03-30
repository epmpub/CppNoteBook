# set password policy



1 script

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

