# Sysmon:(2)服务隐藏

 

## Hide Sysmon from services.msc

Hide:
 sc sdset Sysmon D:(D;;DCLCWPDTSD;;;IU)(D;;DCLCWPDTSD;;;SU)(D;;DCLCWPDTSD;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)
 Restore:
 sc sdset Sysmon D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)

 

 

Hide winlogbeat:

sc sdset winlogbeat D:(D;;DCLCWPDTSD;;;IU)(D;;DCLCWPDTSD;;;SU)(D;;DCLCWPDTSD;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)

 

 

 

sc sdset winlogbeat D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)

 

 

 

 

## Default 1\2\5\16;

 

![计算机生成了可选文字: 2  3  5  6  7  10  11  12  13  14  15  17  18  Tag  ProcessCreate  Fi I eCreateTime  NetworkConnect  ProcessTerminate  DriverLoad  ImageLoad  CreateRemoteThread  RawAccessRead  ProcessAccess  Fi leCreate  RegistryEvent  RegistryEvent  RegistryEvent  Fi I eCreateStreamHash  PipeEvent  PipeEvent  Event  Process Create  File creation time changed  Network connection detected  Process terminated  Driver loaded  Image loaded  CreateRemoteThread detected  RawAccessRead detected  Process accessed  File created  Registry object added or deleted  Registry value set  Registry object renamed  File stream created  Pipe Created  Pipe Connected ](file:///C:/Users/sheng/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)

 

 

## Sysmon: (Version: 10)

 

![计算机生成了可选文字: 川  12  13  15  18  19  Tag  FileCreateTime  NetworkConnect  DriverLoad  ImageLoad  CreateRemoteThread  RawAccessRead  processAccess  FileCreate  RegistryEvent  RegistryEvent  RegistryEvent  FileCreateStrea1THash  PipeEvent  PipeEvent  WmiEvent  WmiEvent  WmiEvent  DnsQuery  Event  process Create  F i 1 巳 creation time changed  Network connection detected  process  Driver loaded  Image loaded  CreateRemoteThread detected  RawAccessRead detected  process accessed  F i 1 巳 created  Registry object added or d 巳 1 巳 t 巳 d  Registry value s 巳 t  Registry object renamed  F i 1 巳 s tr 巳 am created  Pipe Created  Pipe Connected  WITLiEventFilter activity detected  WITLiEventConS1-u-ner activity detected  WITLiEventConSLu-nerToFilter activity detected  Dns query ](file:///C:/Users/sheng/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

## Configuration usage (current schema is version: 3.30):

 

Configuration files can be specified after the -i (installation) or -c (configuration) switches. They make it easier to deploy a preset configuration and to filter captured events.

 

A simple configuration xml file looks like this:

 

![计算机生成了可选文字: (Sysmon  Capture all hashes  K/ HashA1gorithms»  (EventFi1tering»  Log all drivers except if the signature  contains Microsoft or Windows  (Driverl.oad  (Signature  (Signature  K/ DriverLoad»  Do not log process termination  (ProcessTerminate onraatch—" include" / »  Log network connection if the destination port equal  or 80, and process isn't InternetExp10rer  (Network-Connect onraatch—" include '5  K/NetworkConnect»  (Network-Connect  (Image . exe«/lmage»  K/NetworkConnect»  K/ EventFi1tering»  K/ Sysmon»  443 ](file:///C:/Users/sheng/AppData/Local/Temp/msohtmlclip1/01/clip_image003.png)

 

 

The configuration file contains a schemaversion attribute on the Sysmon tag.This version is independent from the Sysmon binary version and allows the parsing of older configuration files. The current schema version is shown in the sample configuration.

 

Configuration entries are directly under the Sysmon tag and filters are under the EventFiltering tag. Configuration entries are similar to command line switches, and have their configuration entry described in the Sysmon usage output. Parameters are optional based on the tag. If a command line switch also enables an event, it needs to be configured though its filter tag.

 

Event filtering allows you to filter generated events. In many cases events can be noisy and gathering everything is not possible. For example, you might be interested about network connections only for a certain process, but not all of them. You can filter the output on the host reducing the data to collect.

 

Each event has its own filter tag under EventFiltering:

 

 

You can also find these tags in the event viewer on the task name.

 

The onmatch filter is applied if events are matched. It can be changed with the "onmatch" attribute for the filter tag. If the value is 'include', it means only matched events are included. If it is set to 'exclude', the event will be included except if a rule match.

 

Each tag under the filter tag is a fieldname from the event.

Each field entry is tested against generated events, if one match the rule is applied and the rest is ignored.

 

For example this rule will discard any process event where the IntegrityLevel is medium:

 

  <ProcessCreate onmatch="exclude">

​    <IntegrityLevel>Medium</IntegrityLevel>

  </ProcessCreate>

 

Field entries can use other conditions to match the value. The conditions areas follow (all are case insensitive):

 

| is         | Default,  values are equals.                         |
| ---------- | ---------------------------------------------------- |
| is not     | Values  are different.                               |
| contains   | The  field contains this value.                      |
| excludes   | The field does not contain this value.               |
| begin with | The field begins with this value.                    |
| end with   | The field ends with this value.                      |
| less than  | Lexicographical comparison is less than  zero.       |
| more than  | Lexicographical comparison is more than  zero.       |
| image      | Match an image path (full path or only image  name). |

​      

 

For example: lsass.exe will match c:\windows\system32\lsass.exe.

 

You can use a different condition by specifying it as an attribute. This excludes network activity from processes with iexplore.exe in their path:

 

 <NetworkConnect onmatch="exclude">

  <Image condition="contains">iexplore.exe</Image>

 </NetworkConnect>

 

You can use both include and exclude rules for the same tag, where exclude rules override include rules.

 

Within a rule, filter conditions on the same field have OR behavior, whereas conditions on different fields have AND behavior. In the sample configuration shown earlier, the networking filter uses both an include and exclude rule to capture activity to port 80 and 443 by all processes except those that have iexplore.exe in their name.

 

##  FAQ：

1. ### Sysmon 15 is not able to start service in timely manner?

   Hi, Same problem with 2012 R2 server, but after cleaned registry, i have tried with Powershell and installation is successful:

   - Remove the reference to the service and driver from the Registry:

     - ```
       Computer\HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Sysmon[64]
       ```

     - `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Sysmon[64]`

   Reboot and delete SysmonDrv.sys from c:\windows folder

   execute in powerShell as administrator:

   PowerShell

   ```powershell
   .\sysmon64.exe -u force
   ```

   Install with:

   PowerShell

   ```powershell
   .\sysmon64.exe -accepteula -i my_config.xml
   ```

   good luck