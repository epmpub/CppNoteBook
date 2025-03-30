# sysmon for linux

## Output

```
sudo tail -f /var/log/syslog
```

or more human-readable

```
sudo tail -f /var/log/syslog | sudo /opt/sysmon/sysmonLogView
```



SysmonLogView has options to filter the output to make it easy to identify specific events or reduce outputted fields for brevity.

SysmonLogView is built when Sysmon is built and is installed into /opt/sysmon when sysmon is installed.

*Important*: You may wish to modify your Syslogger config to ensure it can handle particularly large events (e.g. >64KB, as defaults are often between 1KB and 8KB), and/or use the FieldSizes configuration entry to limit the length of output for some fields, such as CommandLine, Image, CurrentDirectory, etc.

Example:

Add <FieldSizes>CommandLine:100,Image:20</FieldSizes> under <Sysmon> in your configuration file.

## Developer Details