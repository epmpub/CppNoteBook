# Sysmon:2)Get-WinEvent :Use FilterHashtable to filter log

This example uses the **FilterHashtable** parameter to get events from the **Application** log. The hash table uses **key/value** pairs. For more information about the **FilterHashtable** parameter, see [Creating Get-WinEvent queries with FilterHashtable](https://learn.microsoft.com/en-us/powershell/scripting/samples/Creating-Get-WinEvent-queries-with-FilterHashtable). For more information about hash tables, see [about_Hash_Tables](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_hash_tables?view=powershell-7.4).

PowerShell

```powershell
$Date = (Get-Date).AddDays(-2)
Get-WinEvent -FilterHashtable @{ LogName='Application'; StartTime=$Date; Id='1003' }
```

The `Get-Date` cmdlet uses the **AddDays** method to get a date that is two days before the current date. The date object is stored in the `$Date` variable.

The `Get-WinEvent` cmdlet gets log information. The **FilterHashtable** parameter is used to filter the output. The **LogName** key specifies the value as the **Application** log. The **StartTime** key uses the value stored in the `$Date` variable. The **Id** key uses an Event Id value, **1003**.



### Example 18: Use FilterHashtable to get application errors

This example uses the **FilterHashtable** parameter to find Internet Explorer application errors that occurred within the last week.

PowerShell

```powershell
$StartTime = (Get-Date).AddDays(-7)
Get-WinEvent -FilterHashtable @{
  Logname='Microsoft-Windows-Sysmon/Operational'
  ProviderName='Application Error'
  Data='iexplore.exe'
  StartTime=$StartTime
}


// 需要管理员权限操作：

Get-WinEvent -FilterHashtable @{Logname='Microsoft-Windows-Sysmon/Operational';Id='3'}

```

The `Get-Date` cmdlet uses the **AddDays** method to get a date that is seven days before the current date. The date object is stored in the `$StartTime` variable.

The `Get-WinEvent` cmdlet gets log information. The **FilterHashtable** parameter is used to filter the output. The **LogName** key specifies the value as the **Application** log. The **ProviderName** key uses the value, **Application Error**, which is the event's **Source**. The **Data** key uses the value **iexplore.exe** The **StartTime** key uses the value stored in `$StartTime` variable.